#####################################################
#####################################################
###                                               ###
###   Fastfile v0.1.11 by iForests (2017/02/09)   ###
###                                               ###
#####################################################
#####################################################

# Customise this file, documentation can be found here:
# https://github.com/fastlane/fastlane/tree/master/fastlane/docs
# All available actions: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md
# can also be listed using the `fastlane actions` command

# Change the syntax highlighting to Ruby
# All lines starting with a # are ignored when running `fastlane`

# If you want to automatically update fastlane if a new version is available:
# update_fastlane

# This is the minimum version number required.
# Update this, if you use features of a newer version
fastlane_version "2.14.2"

default_platform :android

def get_release_apk_path
  "#{ENV['PWD']}/app/build/outputs/apk/app-release.apk"
end

def get_gradle_path
  "#{ENV['PWD']}/app/build.gradle"
end

def version_name2code(name = nil)
  raise if name.nil?

  major, minor, patch = name.split('.').map(&:to_i)
  patch = 0 if patch.nil?

  major * 1000 + minor * 1 + patch * 0
end

def set_version_code(v = nil)
  raise if v.nil?

  path = get_gradle_path
  re = /versionCode\s+(\d+)/ 

  s = File.read(path)
  versionCode = s[re, 1].to_i

  result = v.to_s
  s[re, 1] = result

  f = File.new(path, 'w')
  f.write(s)
  f.close

  result
end

def set_version_name(v = nil)
  raise if v.nil?

  path = get_gradle_path
  re = /versionName\s+\"([\d\.]+)\"/

  s = File.read(path)
  s[re, 1] = v

  f = File.new(path, 'w')
  f.write(s)
  f.close
end

def bump_version_name(which = nil, show_patch = true)
  raise if which.nil?

  path = get_gradle_path
  re = /versionName\s+\"([\d\.]+)\"/

  s = File.read(path)
  versionName = s[re, 1]
  major, minor, patch = versionName.split('.').map(&:to_i)
    
  raise if minor.nil?
  patch = 0 if patch.nil?

  major += 1 if which == 'major'
  minor += 1 if which == 'minor'
  patch += 1 if which == 'patch'

  result = major.to_s + '.' + minor.to_s
  result += '.' + patch.to_s if show_patch

  s[re, 1] = result

  f = File.new(path, 'w')
  f.write(s)
  f.close

  result
end

def get_version_name
  path = get_gradle_path
  re = /versionName\s+\"([\d\.]+)\"/

  s = File.read(path)
  s[re, 1]
end

platform :android do
  before_all do
    # ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
  end


  lane :resetapk do
    reset_git_repo(
        force: true,
        files: [get_release_apk_path]
    )
  end


  lane :install do
    r = adb(command: "install -r " + get_release_apk_path)
    if r.include? "Failure"
      reason = r[/Failure \[(\w+)\]/, 1]
      if prompt(text: "Install failed (#{reason}). Do you want to uninstall and try again?", boolean: true)
        uninstall
        install
      else
        UI.user_error!(r)
      end
    end
  end

  lane :uninstall do
    adb(command: "uninstall #{CredentialsManager::AppfileConfig.try_fetch_value(:package_name)}")
  end


  lane :clean do
    gradle(task: "clean")
  end

  lane :build do
    ensure_git_status_clean

    default_version_name = get_version_name
    version_name = prompt(text: "Set versionName to (now the value is #{default_version_name}, last git tag is #{last_git_tag}): ")

    set_version_name(version_name)
    set_version_code(version_name2code(version_name))

    clean
    gradle(task: "assembleRelease")

    install
  end


  lane :deploy do |options|
    version_name = get_version_name

    if prompt(text: "Are you sure you want to deploy v#{version_name}?", boolean: true)
      Actions.sh("git add -f #{get_release_apk_path}")
      git_add(path: get_gradle_path)
      git_commit(path: [get_gradle_path, get_release_apk_path],
          message: 'Release version: v' + version_name)

      add_git_tag(tag: 'v' + version_name)
    
      if options[:submit] != false
        supply(
            apk_paths: [get_release_apk_path],
            skip_upload_metadata: true,
            skip_upload_images: true,
            skip_upload_screenshots: true
        )
      end

      new_version = bump_version_name('patch')

      git_commit(path: get_gradle_path,
          message: 'Prepare next development version: v' + new_version)
      push_to_git_remote(local_branch: 'master')
    end
  end


  after_all do |lane|
    # This block is called, only if the executed lane was successful

    # slack(
    #   message: "Successfully deployed new App Update."
    # )
  end

  error do |lane, exception|
    # slack(
    #   message: exception.message,
    #   success: false
    # )
  end
end


# More information about multiple platforms in fastlane: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Platforms.md
# All available actions: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md

# fastlane reports which actions are used
# No personal data is sent or shared. Learn more at https://github.com/fastlane/enhancer
