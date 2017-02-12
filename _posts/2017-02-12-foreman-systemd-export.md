---
layout: post
categories:
- foreman
- ruby
- rails
title: Foreman Systemd Export
description: Using foreman to export startup scripts for systemd
date: 2017-02-12 10:00 AM
---

I switched my side project to a single Linode instance this weekend in an effort to simplify its setup. In the process I decided to try [foreman][foreman] as a way of keeping the rails server up and running. I was very pleased with what came out of it.

### Export scripts

To start you need to export systemd scripts into the `/etc/systemd/system` folder. Make sure to replace `--user deploy` with the user that your app should run as. This capistrano task will handle that.

```ruby
namespace :app do
  desc "Reload systemd"
  task :systemd do
    on roles(:web) do
      within release_path do
        execute :sudo, :foreman, :export, :systemd, "/etc/systemd/system", "--user deploy"
        execute :sudo, :systemctl, "daemon-reload"
      end
    end
  end
end
```

The first deploy you won't need to `daemon-reload`, but whenever a systemd file changes the daemon needs to reload to know about the changes.

The export will create files that are named `app-web@.service`, where `web` is the name of the `Procfile` line. In systemd an `@` service allows you to pass in parameters to the script. For this case, it will be the `PORT` environment variable.

### Start/Stop/Restart

Once the scripts are in place. You need capistrano tasks that will. These tasks are all essentially the same and look like this:

```ruby
namespace :app do
  desc "Start web server"
  task :start do
    on roles(:web) do |host|
      within release_path do
        execute :sudo, :systemctl, :start, "app-web@5000.service"
        execute :sudo, :systemctl, :start, "app-worker@5100.service"
      end
    end
  end
end
```

This will start up a `web` and `worker` instance with the default ports that foreman uses. As you add in new Procfile lines, you will need to add a new line here.

### Adding to a deploy

Finally you need to insert systemd into the deploy cycle. This is easy by adding the tasks in after the appropriate capistrano task `deploy:publishing`.

```ruby
after 'deploy:publishing', 'app:systemd'
after 'app:systemd', 'app:restart'
```

### Enhancements

Capistrano connects into the `deploy` user and that user has sudo rights. It would be more secure to only allow these specific commands to be `sudo`able. Add this to your `sudoers` file to enable specific commands only.

```
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart app-web@5000.service
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart app-worker@5100.service
```

### The full `config/deploy.rb` file

```ruby
namespace :app do
  desc "Start web server"
  task :start do
    on roles(:web) do |host|
      within release_path do
        execute :sudo, :systemctl, :start, "app-web@5000.service"
        execute :sudo, :systemctl, :start, "app-worker@5001.service"
        execute :sudo, :systemctl, :start, "app-clock@5002.service"
      end
    end
  end

  desc "Stop web server"
  task :stop do
    on roles(:web) do |host|
      within release_path do
        execute :sudo, :systemctl, :stop, "app-web@5000.service"
        execute :sudo, :systemctl, :stop, "app-worker@5001.service"
        execute :sudo, :systemctl, :stop, "app-clock@5002.service"
      end
    end
  end

  desc "Restart web server"
  task :restart do
    on roles(:web) do |host|
      within release_path do
        execute :sudo, :systemctl, :restart, "app-web@5000.service"
        execute :sudo, :systemctl, :restart, "app-worker@5001.service"
        execute :sudo, :systemctl, :restart, "app-clock@5002.service"
      end
    end
  end

  desc "Reload systemd"
  task :systemd do
    on roles(:web) do
      within release_path do
        execute :sudo, :foreman, :export, :systemd, "/etc/systemd/system", "--user deploy"
        execute :sudo, :systemctl, "daemon-reload"
      end
    end
  end
end

after 'deploy:publishing', 'app:systemd'
after 'app:systemd', 'app:restart'
```

[foreman]: https://github.com/ddollar/foreman
