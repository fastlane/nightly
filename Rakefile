require 'logger'
$stdout.sync = true
log_path = ENV['FASTLANE_NIGHTLY_LOG_PATH'] ? File.join(ENV['FASTLANE_NIGHTLY_LOG_PATH'], 'nightly.log') : STDOUT
logger = Logger.new(log_path)

gem_name = ENV["GEM_NAME"] || (raise "Please provide a gem name using GEM_NAME")
repo_name = ENV["REPO_NAME"] || gem_name
git_url = ENV["GIT_URL"] || "https://github.com/#{repo_name}/#{repo_name}"
version_file_path = ENV["VERSION_FILE_PATH"] || File.join(gem_name, "lib", gem_name, "version.rb")

desc "Prepare a new fastlane beta build"
task :beta do
  logger.info '*** STARTING NIGHTLY BUILD'
  require 'fastlane'
  require 'shellwords'

  gem_details = Fastlane::PluginManager.fetch_gem_info_from_rubygems(gem_name)
  raise "gem #{gem_name} isn't available online" unless gem_details && gem_details["version"]
  previous_version = Gem::Version.new(gem_details["version"])
  beta_number = Time.now.utc.strftime("%Y%m%d%H%M%S")
  new_version = previous_version.bump.to_s + ".0.beta.#{beta_number}"

  if File.directory?(repo_name)
    logger.info "Pulling from master: #{repo_name}"
    Dir.chdir(repo_name) do
      sh "git fetch origin && git reset --hard origin/master && git pull"
    end
  else
    begin
      logger.info "Cloning repo: #{git_url}"
      sh "git clone --depth 1 #{git_url}"
    rescue => ex
      logger.error ex
      raise "Couldn't clone git repo, you can provide a custom URL using GIT_URL"
    end
  end

  Dir.chdir(repo_name) do
    Fastlane::Actions.load_default_actions
    Fastlane::Actions.load_helpers
    Fastlane::Actions::VersionBumpPodspecAction.run({path: version_file_path, version_number: new_version})

    loger.info(sh "gem list fastlane")

    sh "rake install"
    logger.info "About to deploy #{new_version} to RubyGems"
    gem_path = "./pkg/#{gem_name}-#{new_version}.gem"

    with_api_key do |gem_config_path|
      sh "gem push '#{gem_path}' --config-file #{gem_config_path.shellescape}"
    end

    if ENV["SLACK_URL"]
      logger.info "Posting to Slack"
      # Post a message to Slack about the new release
      config = FastlaneCore::Configuration.create(Fastlane::Actions::SlackAction.available_options, {
        message: "Successfully pushed new nightly build `#{new_version}` :rocket:\n\nPlease give it a shot using\n\n`gem update fastlane --pre`\n\nor by adding\n\n`gem 'fastlane', '>= #{new_version}'`\n\nto your Gemfile",
        channel: ENV["SLACK_CHANNEL"] || "releases",
        default_payloads: []
      })
      Fastlane::Actions::SlackAction.run(config)
    end

    logger.info "Successfully deployed #{new_version} to RubyGems"
  end
  logger.info "Cleaning up old gems"
  sh "gem cleanup"
  loger.info(sh "gem list fastlane")
end

def with_api_key
  require 'tempfile'

  raise "No Rubygems API key provided" if ENV["RUBYGEMS_API_KEY"].to_s.length == 0
  raise "Please provide block" unless block_given?

  gem_config = Tempfile.new('rubygems')
  gem_config.write("---\n:rubygems_api_key: #{ENV['RUBYGEMS_API_KEY']}")
  gem_config.rewind

  yield(gem_config.path.strip)
  gem_config.close
  gem_config.unlink
end
