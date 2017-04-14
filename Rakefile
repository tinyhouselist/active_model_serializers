# frozen_string_literal: true
require 'English'
require 'rake/clean'

GEMSPEC = 'ams.gemspec'
require File.join(File.dirname(__FILE__), 'lib', 'ams', 'version')
VERSION = AMS::VERSION
VERSION_TAG = "AMS_#{VERSION}"
GEMPATH = "AMS-#{VERSION}.gem"
CLEAN.include(GEMPATH)
CLOBBER << GEMPATH

SOURCE_FILES = Rake::FileList.new(GEMSPEC)

rule '.gem' => '.gemspec' do
  sh "gem build -V #{GEMSPEC}"
end

desc 'build gem'
task build: [:clobber, SOURCE_FILES.ext('.gem')]

desc 'install gem'
task install: :build do
  sh "gem install #{GEMPATH}"
end

desc 'uninstall gem'
task :uninstall do
  sh 'gem uninstall -aIx AMS'
end

desc 'test install message'
task :test_install do
  gemspec = Gem::Specification.load(GEMSPEC)
  puts "You should see post install message '#{gemspec.post_install_message}' below:"
  begin
    Rake::Task['install'].invoke
  ensure
    Rake::Task['uninstall'].invoke
  end
  puts "#{GEMSPEC} => #{GEMPATH}"
end

desc 'abort when repo not clean or has uncommited code'
task :assert_clean_repo do
  sh 'git diff --exit-code'
  abort 'Git repo not clean' unless $CHILD_STATUS.success?
  sh 'git diff-index --quiet --cached HEAD'
  abort 'Git repo not commited' unless $CHILD_STATUS.success?
end

task push_and_tag: [:build] do
  sh "gem push #{GEMPATH}"
  if $CHILD_STATUS.success?
    Rake::Task['tag_globally'].invoke
  else
    abort 'tagging aborted; pushing gem failed.'
  end
end

task :tag_globally do
  sh "git tag -a -m \"Version #{VERSION}\" #{VERSION_TAG}"
  STDOUT.puts "Tagged #{VERSION_TAG}."
  sh 'git push'
  sh 'git push --tags'
end

desc 'Release'
task release: [:assert_clean_repo, :push_and_tag]

import('lib/tasks/rubocop.rake')
import('lib/tasks/doc.rake')
import('lib/tasks/test.rake')

task default: [:test, :rubocop]

desc 'CI test task'
task ci: [:default, 'doc:stats']
