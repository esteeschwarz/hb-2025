# frozen_string_literal: true

###############################################################################
#
# CollectionBuilder Rake Utilities
#
# See "docs/rake_tasks/" for documentation.
# See "rakelib" for individual rake tasks!
#
###############################################################################

require 'csv'
require 'fileutils'
require 'mini_magick'  # Add to top with other requires

###############################################################################
# Helper Functions
###############################################################################

def prompt_user_for_confirmation(message)
  response = nil
  loop do
    print "#{message} (Y/n): "
    $stdout.flush
    response = case $stdin.gets.chomp.downcase
               when '', 'y' then true
               when 'n' then false
               end
    break unless response.nil?

    puts 'Please enter "y" or "n"'
  end
  response
end
###################
# require 'mini_magick'  # Add to top with other requires
#require 'fileutils'
require 'mini_magick'
#require 'fileutils'

namespace :gender do
  # Configuration - adjust these paths
  PDF_SOURCE_DIR = "objects"
  OUTPUT_DIR = "objects/derivatives"
  THUMBNAIL_SIZE = "600x600"
  FULL_SIZE = "1200x1200>"
  DENSITY = 150

  desc "Generate PDF derivatives (works without Ghostscript)"
  task :pdf do
    # Create output directories
    FileUtils.mkdir_p("#{OUTPUT_DIR}/thumb")
    FileUtils.mkdir_p("#{OUTPUT_DIR}/full")

    # Try to find available converters
    converters = []
    converters << :pdftoppm if system("which pdftoppm > /dev/null 2>&1")
    converters << :imagemagick if system("which convert > /dev/null 2>&1")
    converters << :pdftocairo if system("which pdftocairo > /dev/null 2>&1")

    if converters.empty?
      abort "No PDF conversion tools found. Install one of:\n" +
            "  brew install poppler (for pdftoppm/pdftocairo)\n" +
            "  brew install imagemagick (for convert)"
    end

    puts "Available converters: #{converters.join(', ')}"

    Dir.glob("#{PDF_SOURCE_DIR}/*.pdf").each do |pdf|
      base = File.basename(pdf, '.*')
      puts "\nProcessing: #{base}.pdf"

      # Thumbnail generation
      thumb_out = "#{OUTPUT_DIR}/thumb/#{base}.jpg"
      unless File.exist?(thumb_out)
        success = false
        converters.each do |converter|
          break if success
          
          puts "  Trying #{converter}..."
          case converter
          when :pdftoppm
            success = system(
              "pdftoppm -jpeg -f 1 -l 1 -r #{DENSITY} -scale-to #{THUMBNAIL_SIZE.split('x').first} " +
              "-singlefile '#{pdf}' '#{OUTPUT_DIR}/thumb/#{base}_thumb'"
            )
          when :imagemagick
            success = system(
              "convert -density #{DENSITY} '#{pdf}[0]' " +
              "-thumbnail '#{THUMBNAIL_SIZE}' " +
              "-background white -alpha remove " +
              "-quality 85 '#{thumb_out}'"
            )
          when :pdftocairo
            success = system(
              "pdftocairo -jpeg -f 1 -l 1 -scale-to #{THUMBNAIL_SIZE.split('x').first} " +
              "'#{pdf}' '#{thumb_out}'"
            )
          end
        end
        puts success ? "  ✓ Thumbnail created" : "  ✗ All converters failed for thumbnail"
      end

      # Full-size generation
      full_out = "#{OUTPUT_DIR}/full/#{base}.jpg"
      unless File.exist?(full_out)
        success = false
        converters.each do |converter|
          break if success
          
          puts "  Trying #{converter}..."
          case converter
          when :pdftoppm
            success = system(
              "pdftoppm -jpeg -f 1 -l 1 -r #{DENSITY} -scale-to #{FULL_SIZE.split('x').first} " +
              "-singlefile '#{pdf}' '#{OUTPUT_DIR}/full/#{base}_full'"
            )
          when :imagemagick
            success = system(
              "convert -density #{DENSITY} '#{pdf}[0]' " +
              "-resize '#{FULL_SIZE}' " +
              "-background white -alpha remove " +
              "-quality 90 '#{full_out}'"
            )
          when :pdftocairo
            success = system(
              "pdftocairo -jpeg -f 1 -l 1 -scale-to #{FULL_SIZE.split('x').first} " +
              "'#{pdf}' '#{full_out}'"
            )
          end
        end
        puts success ? "  ✓ Full-size created" : "  ✗ All converters failed for full-size"
      end
    end
  end
end