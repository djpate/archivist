require 'bundler'
require 'rake/testtask'
Bundler::GemHelper.install_tasks

Rake::TestTask.new do |t|
  t.libs << 'lib'
  t.test_files = FileList['test/test*.rb']
  t.verbose = true
end