require "rubygems"

$: << File.expand_path("../lib", __FILE__)

require "subdb-sync/version"

task :default => [:test]

desc "Run tests"
task :test do
  $: << File.expand_path("../test", __FILE__)

  Dir["./test/**/*_test.rb"].each do |test|
    require test
  end
end

desc "Run Swing UI for testing"
task :swing do
  $: << File.expand_path("..", __FILE__)

  require 'subdb/ui/swing'
end

desc "Build jar"
task :build do
  require "bundler"

  `mkdir -p releases`

  unless File.exists?("vendor/jruby-complete-1.6.1.jar")
    puts "Downloading jruby-complete-1.6.1.jar, it may take a while..."
    `curl -o vendor/jruby-complete-1.6.1.jar http://jruby.org.s3.amazonaws.com/downloads/1.6.1/jruby-complete-1.6.1.jar`
  end

  `rm -rf jarbuild` if File.directory?("jarbuild")
  `mkdir -p jarbuild`

  Bundler.load.specs.each do |s|
    next if s.name == "bundler"

    puts "Vendorizing #{s.name} for jarbuild..."
    `cp -rf #{s.full_gem_path}/lib/* jarbuild`
  end

  puts "Copying files to jarbuild..."
  `cp -rf lib/* jarbuild`
  `cp -rf javalib jarbuild`
  `cp -rf images jarbuild`

  Dir.chdir("jarbuild") do
    files = Dir.glob("./*")

    `cp ../vendor/jruby-complete-1.6.1.jar subdb.jar`
    `cp ../lib/subdb-sync/swing.rb jar-bootstrap.rb`

    puts "Adding files to jar..."
    `jar uf subdb.jar #{files.join(' ')}`

    puts "Setting up jar initialization..."
    `jar ufe subdb.jar org.jruby.JarBootstrapMain jar-bootstrap.rb`
  end

  jarname = "releases/subdb-#{Subdb::VERSION}.jar"

  `rm #{jarname}` if File.exists?(jarname)
  `mv jarbuild/subdb.jar #{jarname}`
  `rm -rf jarbuild`

  puts "Done building #{jarname}"
end

# requires Ant JarBundler: http://www.informagen.com/JarBundler/
desc "Build mac app"
task :mac_app => :build do
  puts "Building mac package..."
  `ant -Dversion=#{Subdb::VERSION}`
end

desc "Build mac dist"
task :build_mac => :mac_app do
  puts "Building dmg file..."
  `hdiutil create releases/subdb-#{Subdb::VERSION}.dmg -ov -srcfolder releases/SubDB.app`
  `rm -rf releases/SubDB.app`

  puts "Done build mac dist releases/subdb-#{Subdb::VERSION}.dmg"
end
