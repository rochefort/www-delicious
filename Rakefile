require "rubygems"
require "rake/testtask"
require "rake/gempackagetask"
begin
  require "hanna/rdoctask"
rescue LoadError
  require "rake/rdoctask"
end

$:.unshift(File.dirname(__FILE__) + "/lib")
require "www/delicious"


# Common package properties
PKG_NAME    = ENV['PKG_NAME']    || WWW::Delicious::GEM
PKG_VERSION = ENV['PKG_VERSION'] || WWW::Delicious::VERSION
RUBYFORGE_PROJECT = "www-delicious"

if ENV['SNAPSHOT'].to_i == 1
  PKG_VERSION << "." << Time.now.utc.strftime("%Y%m%d%H%M%S")
end


# Run all the tests in the /test folder
Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList["test/**/*_test.rb"]
  t.verbose = true
end

# Generate documentation
Rake::RDocTask.new do |rd|
  rd.main = "README.rdoc"
  rd.rdoc_files.include("*.rdoc", "lib/**/*.rb")
  rd.rdoc_dir = "rdoc"
end

# Run test by default.
task :default => ["test"]


# This builds the actual gem. For details of what all these options
# mean, and other ones you can add, check the documentation here:
#
#   http://rubygems.org/read/chapter/20
#
spec = Gem::Specification.new do |s|

  s.name              = PKG_NAME
  s.version           = PKG_VERSION
  s.summary           = "Ruby client for delicious.com API."
  s.author            = "Simone Carletti"
  s.email             = "weppos@weppos.net"
  s.homepage          = "http://www.simonecarletti.com/code/www-delicious"
  s.description       = <<-EOD
    WWW::Delicious is a del.icio.us API client implemented in Ruby. \
    It provides access to all available del.icio.us API queries \
    and returns the original XML response as a friendly Ruby object.
  EOD
  s.rubyforge_project = RUBYFORGE_PROJECT

  s.has_rdoc          = true
  # You should probably have a README of some kind. Change the filename
  # as appropriate
  s.extra_rdoc_files  = Dir.glob("*.rdoc")
  s.rdoc_options      = %w(--main README.rdoc)

  # Add any extra files to include in the gem (like your README)
  s.files             = %w() + Dir.glob("*.{rdoc,gemspec}") + Dir.glob("{lib,test}/**/*")
  s.require_paths     = ["lib"]

  # If you want to depend on other gems, add them here, along with any
  # relevant versions
  # s.add_dependency("some_other_gem", "~> 0.1.0")

  # If your tests use any gems, include them here
  s.add_development_dependency("mocha")
end

# This task actually builds the gem. We also regenerate a static
# .gemspec file, which is useful if something (i.e. GitHub) will
# be automatically building a gem for this project. If you're not
# using GitHub, edit as appropriate.
Rake::GemPackageTask.new(spec) do |pkg|
  pkg.gem_spec = spec
end

desc "Build the gemspec file #{spec.name}.gemspec"
task :gemspec do
  file = File.dirname(__FILE__) + "/#{spec.name}.gemspec"
  File.open(file, "w") {|f| f << spec.to_ruby }
end

desc "Remove any temporary products, including gemspec"
task :clean => [:clobber] do
  rm "#{spec.name}.gemspec" if File.file?("#{spec.name}.gemspec")
end

desc "Remove any generated file"
task :clobber => [:clobber_rdoc, :clobber_rcov, :clobber_package]

desc "Package the library and generates the gemspec"
task :package => [:gemspec]

begin
  require "rcov/rcovtask"

  desc "Create a code coverage report"
  Rcov::RcovTask.new do |t|
    t.test_files = FileList["test/**/*_test.rb"]
    t.ruby_opts << "-Itest -x mocha,rcov,Rakefile"
    t.verbose = true
  end
rescue LoadError
  task :clobber_rcov
  puts "RCov is not available"
end

begin
  require "code_statistics"
  desc "Show library's code statistics"
  task :stats do
    CodeStatistics.new(["WWW::Delicious", "lib"],
                       ["Tests", "test"]).to_s
  end
rescue LoadError
  puts "CodeStatistics (Rails) is not available"
end

desc "Publish documentation to the site"
task :publish_rdoc => [:clobber_rdoc, :rdoc] do
  ENV["username"] || raise(ArgumentError, "Missing ssh username")
  sh "rsync -avz --delete rdoc/ #{ENV["username"]}@code:/var/www/apps/code/#{PKG_NAME}/api"
end

desc "Open an irb session preloaded with this library"
task :console do
  sh "irb -rubygems -I lib -r www/delicious.rb"
end


Dir["tasks/**/*.rake"].each do |file|
  load(file)
end
