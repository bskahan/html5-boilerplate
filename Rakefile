require 'rake/clean'
require 'jsmin'
require 'cssmin'

CLOBBER.include('publish')
EXCLUDE = %w{Rakefile .gitignore README.markdown .git build test demo publish}

directory 'publish'
directory 'publish/css'
directory 'publish/images'
directory 'publish/js'

task :default => :pub

desc 'Copy over files to ./publish'
task :pub => ['publish', *FileList['*'].exclude(*EXCLUDE).gsub(/^/, 'publish/')]

task :min => ['publish/js/lib.min.js', 'publish/css/style.min.css']

def publication_rule(file_regex, &block)
  action = block_given? ? block : Proc.new { |t| cp t.prerequisites.first, t.name }
  rule(%r{^publish/#{file_regex}$} => Proc.new { |n| n.sub('publish/', '') }, &action)
end

file 'publish/js/lib.min.js' => FileList['js/*.js'] do |t|
  File.open(t.name, 'w') do |f|
    f.write JSMin.minify(t.prerequisites.map { |p| File.read(p) }.join)
  end
end

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