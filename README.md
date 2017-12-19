# Nightly RubyGem beta builds

The scripts in this repo will run on a server to automatically upload new nightly builds to RubyGems. Checkout [Rakefile](Rakefile) for more information.

The script works with any gem, we use it for [fastlane](https://fastlane.tools).

You have to provide the following environment variables:

- `GEM_NAME` the name of your gem
- `RUBYGEMS_API_KEY` the RubyGems API key you can get from [RubyGems.org](https://rubygems.org/profile/edit)

Optional environment variable

- `REPO_NAME` (defaults to `GEM_NAME`)
- `GIT_URL` (defaults to `https://github.com/[repo_name]/[repo_name]`)
- `VERSION_FILE_PATH` (defaults to `File.join(gem_name, "lib", gem_name, "version.rb")`)
- `SLACK_URL` (if you want new releases to be posted to Slack)
- `SLACK_CHANNEL` (only used in combination with `SLACK_URL`, defaults to `"releases"`)

Put all of that on any server (e.g. Heroku) and use a schedule to call `rake beta` every night.

Please note that the nightly-build system run by the `fastlane` core team does not automatically deploy code changes made to this repository. When merging changes to the repository, please let a member of the `fastlane` core team know (via Slack or at-mention in the GitHub pull request), so that they can manually deploy your code change.
