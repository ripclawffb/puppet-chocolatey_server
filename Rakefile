require 'puppetlabs_spec_helper/rake_tasks'
require 'puppet/version'
require 'puppet/vendor/semantic/lib/semantic' unless Puppet.version.to_f < 3.6
require 'puppet-lint/tasks/puppet-lint'
require 'puppet-syntax/tasks/puppet-syntax'

# These gems aren't always present, for instance
# on Travis with --without development
begin
  require 'puppet_blacksmith/rake_tasks'
rescue LoadError
end

Rake::Task[:lint].clear

PuppetLint.configuration.relative = true
# These lint exclusions are in puppetlabs_spec_helper but needs a version above 0.10.3
# Line length test is 80 chars in puppet-lint 1.1.0
PuppetLint.configuration.send("disable_80chars")
# Line length test is 140 chars in puppet-lint 2.x
PuppetLint.configuration.send('disable_140chars')
PuppetLint.configuration.log_format = "%{path}:%{linenumber}:%{check}:%{KIND}:%{message}"
PuppetLint.configuration.fail_on_warnings = true

# Forsake support for Puppet 2.6.2 for the benefit of cleaner code.
# http://puppet-lint.com/checks/class_parameter_defaults/
PuppetLint.configuration.send('disable_class_parameter_defaults')
# http://puppet-lint.com/checks/class_inherits_from_params_class/
PuppetLint.configuration.send('disable_class_inherits_from_params_class')

exclude_paths = [
  "bundle/**/*",
  "pkg/**/*",
  "vendor/**/*",
  "spec/**/*",
]
PuppetLint.configuration.ignore_paths = exclude_paths
PuppetSyntax.exclude_paths = exclude_paths

desc "Run acceptance tests"
RSpec::Core::RakeTask.new(:acceptance) do |t|
  t.pattern = 'spec/acceptance'
end

task :metadata_lint do
  sh "metadata-json-lint metadata.json"
end

desc "Custom: Download third-party modules"
task :r10k do
  system 'r10k puppetfile install -v'
end

desc "Custom: Prepare symbolic link to root puppet module folders"
task :symlink do
  FileUtils.remove_dir("development/modules/chocolatey_server",:verbose => true)
  FileUtils.mkdir_p "development/modules/chocolatey_server"
  FileUtils.cd "development/modules/chocolatey_server"
  system 'ln -s "../../../manifests" "manifests"'
end

desc "Run syntax, lint, and spec tests."
task :test => [
  :syntax,
  :lint,
  :spec,
  :metadata_lint,
  :r10k,
  :symlink
]

task :default => [:test]
