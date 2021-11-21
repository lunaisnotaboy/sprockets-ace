require 'json'
require 'rake/clean'

directory 'tmp/'
CLEAN.include 'tmp/'

file 'tmp/ace' => 'tmp/' do
  cd 'tmp/' do
    sh 'git clone https://github.com/ajaxorg/ace.git'
  end
end

desc 'Build the latest Ace'
task :build => 'tmp/ace' do
  cd 'tmp/ace' do
    sh 'git pull'
    sh 'npm install'
    sh 'node ./Makefile.dryice.js -nc'
  end
end

file 'bower.json' => :build do |f|
  version = nil

  cd 'tmp/ace' do
    version = `git describe --tags`.chomp
    major, minor, sha = version.sub(/^v/, '').split('-')
    version = "#{major}-#{minor}"
  end

  bower = {
    'name' => 'ace',
    'version' => version,
    'main' => './ace.js'
  }

  File.open(f.name, 'w') do |io|
    io.write bower.to_json
  end
end
CLOBBER.include 'bower.json'

file 'ace.js' => :build do |f|
  cp 'tmp/ace/build/src-noconflict/ace.js', f.name
end
CLOBBER.include 'ace.js'

%w(keybinding mode theme ext worker).each do |mod|
  directory "#{mod}/"
  task mod => [:build, "#{mod}/"] do |t|
    Dir["tmp/ace/build/src-noconflict/#{t.name}-*.js"].each do |src|
      cp src, "#{t.name}/#{File.basename(src).sub("#{t.name}-", '')}"
    end
  end
  CLOBBER.include "#{mod}"
end

desc 'Push up the latest Ace to the repo'
task :publish => :copy do
  head = tag = nil

  cd 'tmp/ace' do
    head = `git rev-parse HEAD`.chomp
    tag = `git describe --tags`.chomp
    major, minor, sha = tag.sub(/^v/, '').split('-')
    tag = "#{major}-#{minor}"
  end

  sh 'git add .'
  sh "git commit -m \"Upgrade Ace to #{head}\""
  sh "git tag -s #{tag}"

  sh 'git push'
  sh 'git push --tags'
end

task :copy => ['bower.json', 'ace.js', :keybinding, :mode, :theme, :ext, :worker]
task :default => [:clobber, :copy, :publish]
