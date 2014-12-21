# Load bundler gems
require 'rubygems'
require 'bundler/setup'
Bundler.require(:default)
require 'csv'
require 'mechanize'
require 'open-uri'
require 'rubygems'
require 'zip'
require 'fileutils'

task :scrape_image, :url do |t, args|
  base_url = "https://openartyeg.crowdmap.com/reports/view/"
  agent = Mechanize.new

  count = 0
  CSV.foreach("crowdmap.csv", {:headers=>:first_row, :encoding => 'windows-1251:utf-8'}) do |row|
    count += 1
    report_id = row[0]
    report_url = "#{base_url}#{report_id}"

    page = agent.get report_url
    # Why is this necessary
    page = agent.get report_url
    element = page.search('a.photothumb')
    next if element.empty?
    image_url = element.attribute('href').value
    puts "Downloading report image #{report_id} from url: #{image_url}"
    download_image(image_url, "images/#{report_id}.jpg")
  end

  puts "Creating zip file..."
  directoryToZip = "images"
  outputFile = "out.zip"
  zf = ZipFileGenerator.new(directoryToZip, outputFile)
  zf.write
  puts "Removing image files..."
  FileUtils.rm_rf("images/.", secure: true)
end

def download_image(url, path)
  File.open(path, "wb") do |saved_file|
    # the following "open" is provided by open-uri
    open(url, "rb") do |read_file|
      saved_file.write(read_file.read)
    end
  end
end

# This is a simple example which uses rubyzip to
# recursively generate a zip file from the contents of
# a specified directory. The directory itself is not
# included in the archive, rather just its contents.
#
# Usage:
# require /path/to/the/ZipFileGenerator/Class
# directoryToZip = "/tmp/input"
# outputFile = "/tmp/out.zip"
# zf = ZipFileGenerator.new(directoryToZip, outputFile)
# zf.write()

class ZipFileGenerator
  # Initialize with the directory to zip and the location of the output archive.
  def initialize(inputDir, outputFile)
    @inputDir = inputDir
    @outputFile = outputFile
  end
  # Zip the input directory.
  def write()
    entries = Dir.entries(@inputDir); entries.delete("."); entries.delete("..")
    io = Zip::File.open(@outputFile, Zip::File::CREATE);
    writeEntries(entries, "", io)
    io.close();
  end
  # A helper method to make the recursion work.
  private
  def writeEntries(entries, path, io)
    entries.each { |e|
      zipFilePath = path == "" ? e : File.join(path, e)
      diskFilePath = File.join(@inputDir, zipFilePath)
      puts "Deflating " + diskFilePath
      if File.directory?(diskFilePath)
        io.mkdir(zipFilePath)
        subdir =Dir.entries(diskFilePath); subdir.delete("."); subdir.delete("..")
        riteEntries(subdir, zipFilePath, io)
      else
        io.get_output_stream(zipFilePath) { |f| f.puts(File.open(diskFilePath, "rb").read())}
      end
    }
  end
end
