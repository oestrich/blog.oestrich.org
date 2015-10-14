---
layout: post
categories:
- ruby
- imagemagick
title: Detecting Primary Color in Photo
---

For one of our clients at [SmartLogic][smartlogic], we had to find the primary color of a photo to use as a background while the photo was loading. This was pretty easy given ImageMagick and lot of other tutorials online, but the primary color was always some extremely dark color, or gray/black.

This [sidekiq][sidekiq] worker detects the "best" primary color of the photo, skipping really dark colors and any version of gray.

This uses the `convert` imagemagick command to find the color histogram and sorts it by most used color. It then strips out "bad" colors to find the most common color we can use.

```ruby
class ProcessPhotoWorker
  include Sidekiq::Worker

  # Regex for the output of image magic
  COLOR_REGEX =
     /\d+: \(\s{0,2}\d{1,3},\s{0,2}\d{1,3},\s{0,2}\d{1,3}\) (?<color>#.{6}).*/
  # These are all very dark colors
  COLORS_TO_IGNORE = [
    "#000000", "#000033", "#000066",
    "#003300", "#003333", "#003366",
    "#330000", "#330033", "#330066",
    "#333300", "#333333", "#333366",
    "#660000", "#660033", "#660066",
    "#663300", "#663333", "#663366",
  ]

  def perform(photo_id)
    @photo = Photo.find(photo_id)

    # Save a local version of the S3 photo
    tmpfile = Tempfile.new("photo")
    begin
      tmpfile.binmode
      tmpfile.write(@photo.image.read)
      tmpfile.rewind

      image = ::MiniMagick::Image.read(tmpfile.read)

      capture_geometry(image)
      capture_main_color(image, tmpfile)
    ensure
      tmpfile.close
      tmpfile.unlink
    end

    @photo.save
  end

  private

  def capture_geometry(image)
    @photo.image_height = image[:height]
    @photo.image_width = image[:width]
  end

  # Uses the convert tool to find the most common color
  # Output is similar to:
  #   67655: ( 50, 18, 18) #321212 srgb(50,18,18)
  #  240295: ( 15,  3,  6) #0F0306 srgb(15,3,6)
  def capture_main_color(image, tmpfile)
    output = image.run(command_to_run(tmpfile))

    sorted_colors = output_to_colors(output)
    sorted_colors = sorted_colors - COLORS_TO_IGNORE
    sorted_colors = skip_gray(sorted_colors)

    @photo.image_main_color = sorted_colors.last
  end

  def command_to_run(tmpfile)
    command = ::MiniMagick::CommandBuilder.new("convert")
    command << tmpfile.path << "-resize" << "100x100" << "+dither"
    command << "-remap" << "netscape:" << "-format" << "'%c'" << "histogram:info:"
    command
  end

  def output_to_colors(command_output)
    lines = command_output.split("\n")

    sorted_lines = lines.map do |line|
      # convert white space to single space inbetween for the regex
      # first line contains a "'"
      line.gsub(/\s+/, " ").gsub("'", "").strip
    end.sort_by(&:to_i)

    sorted_colors = sorted_lines.map do |color|
      match = COLOR_REGEX.match(color)
      match[:color] if match
    end

    sorted_colors.reject(&:blank?)
  end

  def skip_gray(sorted_colors)
    sorted_colors.reject do |color|
      color = Color::RGB.from_html(color)
      color.red == color.green &&
        color.red == color.blue &&
        color.green == color.blue
    end
  end
end
```

[smartlogic]: http://smartlogic.io
[sidekiq]: https://github.com/mperham/sidekiq
