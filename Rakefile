# The Rake file provides tasks to build the publication-ready version of the website.
# It compresses code, removes clutter and arranges files in ./publish ready for
# uploading to the server.
#
# INSTALLATION
#
# Make sure your system is set up with Ruby. Rake should be installed by default.
# Then make sure you have all dependencies installed:
#
#   $ gem install cssmin jsmin
#
# You may need to prefix that with 'sudo'.
#
# USAGE
#
# You can see a list of available tasks on your command line:
#
#   $ rake -T
# 
# Each relevant task has a short description.
#
# Run a task by calling its name:
#
#   $ rake min
# 
# When run without any arguments it will invoke the default task, i.e. 'pub'
#
#   $ rake
# 
# DESIGN
#
# The design of these tasks is slightly different from the Ant tasks it is based
# on, in Paul Irish' original project (look it up on github).
#
# When generating the output, all files will be copied and all optimizations will
# be applied. There is no switch to turn, say, javascript compression on and off.
#
# Files will only be copied when they have changed. In fact, all tasks are
# file-based. The 'pub' and 'min' tasks simply invoke the generation of the
# published versions of all found project files.
#
# Compression and minification of assets (javascript and CSS files) is handled
# by simple Ruby gems, not by Yahoo's YUI compressor in java.
#
# CREDITS
#
# Author: Arjan van der Gaag <arjan@arjanvandergaag.nl>
# Date: 2010-08-30
require 'rake/clean'
require 'jsmin'
require 'cssmin'

# When calling the clobber task, this is what will be removed.
CLOBBER.include('publish')

# These files will not be copied into the ./publish directory.
EXCLUDE = %w{Rakefile .gitignore README.markdown .git build test demo publish}

# List the directories that should be created as needed.
directory 'publish'
directory 'publish/css'
directory 'publish/images'
directory 'publish/js'

# Running just 'rake' without any arguments will call the 'pub' task
task :default => :pub

desc 'Copy over files to ./publish'
task :pub => ['publish', *FileList['*'].exclude(*EXCLUDE).gsub(/^/, 'publish/')]

desc 'Generate the minified javascript and CSS files'
task :min => ['publish/js/lib.min.js', 'publish/css/style.min.css']

# Define a rule for publishing files
#
# This is a simple helper function that will define a rule for a publication file
# based on an original file. The given block defines how the file is created.
# Omitting the block will default to simply copying the source file to the target
# location.
def publication_rule(file_regex, &block)
  action = block_given? ? block : Proc.new { |t| cp t.prerequisites.first, t.name }
  rule(%r{^publish/#{file_regex}$} => Proc.new { |n| n.sub('publish/', '') }, &action)
end

# Create the single javascript file by concatenating and minifying all javascript files
file 'publish/js/lib.min.js' => FileList['js/*.js'] do |t|
  File.open(t.name, 'w') do |f|
    f.write JSMin.minify(t.prerequisites.map { |p| File.read(p) }.join)
  end
end

# Create the single stylesheet by minifying the source stylesheet
file 'publish/css/style.min.css' => 'css/style.css' do |t|
  File.open(t.name, 'w') do |f|
    f.write CSSMin.minify(t.prerequisites.map { |p| File.read(p) }.join)
  end
end

# Simply copy files with these extensions
publication_rule(/.+\.(xml|ico|conf|txt|config)/)

publication_rule(/.+\.png/) do |t|
  puts "Optimizing #{t.name}"
  # TODO: do some optimization here
  cp t.prerequisites.first, t.name
end

publication_rule(/.+\.jpg/) do |t|
  puts "Optimizing #{t.name}"
  # TODO: do some optimization here
  cp t.prerequisites.first, t.name
end

# Publish HTML files by copying optimized versions.
#
# This removes development scripts and comments and switches to
# minified versions of jquery.
publication_rule(/.+\.html/) do |t|
  puts "Generating #{t.name}"
  contents = File.read(t.prerequisites.first)
  File.open(t.name, 'w') do |f|
    f.write contents. gsub(/<!-- scripts concatenated [\d\w\s\W]*?!-- end concatenated and minified scripts-->/m, '<script src="js/script.min.js"></script>').
                      gsub(/<!-- yui profiler[\d\w\s\W]*?end profiling code -->/m, '').
                      gsub(/<!-- [\d\w\s\W]*?-->/m, '').
                      gsub(/<!--!/, '<!--').
                      gsub(/jquery-(\d|\d(\.\d)+)\.js/,  'jquery-\1.min.js').
                      gsub(/(\d|\d(\.\d)+)\/jquery\.js/, '\1/jquery.min.js')
  end
end
