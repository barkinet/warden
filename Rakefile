require 'json'
require 'fileutils'
require 'mkmf'
require 'rake/clean'

include FileUtils

def package_json
 @package_json ||= JSON.parse(File.read('package.json'))
end

def version
  package_json['version']
end

def name
  package_json['name']
end

def description
  package_json['description']
end

def license
  package_json['license']
end

def homepage
  package_json['homepage']
end

def target_version
  ::File.read(::File.join(@base_dir, '.nvmrc')).strip()
end

def max_version
   target_version.split('.').first.delete('v').to_f + 1
end

def install_dir
  ::File.join('pkg', 'opt', name)
end

def config_dir
  ::File.join(install_dir, 'config')
end

def base_dir
  @base_dir ||= File.dirname(File.expand_path(__FILE__))
end

task :install do
  sh 'npm install --production'
  # This is required because the conditional package bundles a devDependency
  # that bundles conditional and causes shrinkwrap to complain
  sh 'npm prune --production'
end

task :shrinkwrap => [:install] do
  sh 'npm shrinkwrap'
end

task :pack => [:shrinkwrap] do
  sh 'npm pack'
end

task :package_dirs do
  mkdir_p ::File.join(base_dir, install_dir)
  mkdir_p ::File.join(base_dir, config_dir)
end

task :warden_source => [:install] do
  ['bin/', 'lib/', 'node_modules/', 'LICENSE', 'package.json'].each do |src|
    cp_r ::File.join(base_dir, src), ::File.join(base_dir, install_dir)
  end
  cp ::File.join(base_dir, 'config', 'defaults.json'), ::File.join(base_dir, config_dir)
end


task :chdir_pkg => [:package_dirs] do
  cd ::File.join(base_dir, 'pkg')
end

task :deb => [:chdir_pkg, :warden_source] do
  command = [
    'fpm',
    '--deb-no-default-config-files',
    "--depends \"nodejs >= #{target_version}\"",
    "--depends \"nodejs << #{max_version}\"",
    "--license \"#{license}\"",
    "--url \"#{homepage}\"",
    "--description \"#{description}\"",
    '-s dir',
    '-t deb',
    "-n \"#{name}\"",
    "-v #{version}",
    'opt/'
    ].join(' ')
  sh command
end

desc "Package #{name}"
task :package => [:install, :shrinkwrap, :pack, :deb]

desc "Release #{name} and prepare to create a release on github.com"
task :release do
  puts
  puts "Create a new #{version} release on github.com and upload the #{name} tarball"
  puts 'You can find directions here: https://github.com/blog/1547-release-your-software'
  puts
  puts 'Make sure you add release notes!'
end

CLEAN.include 'npm-shrinkwrap.json'
CLEAN.include "#{name}-*.tgz"
CLEAN.include 'pkg/'
CLEAN.include '**/.DS_Store'
CLEAN.include 'node_modules/'

task :default => [:clean, :package, :release]
