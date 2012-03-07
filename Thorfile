# encoding: utf-8

require 'chef/cookbook/metadata'

class Default < Thor
  desc "release", "Create tag v1.6.2 and build and push twitter-1.6.2.gem to Rubygems"
  method_option :knife_config, :type => :string, :aliases => "-c", :desc => "", :default => "~/.chef/knife.rb"
  def release
    unless clean?
      say "There are files that need to be committed first.", :red
      exit 1
    end

    tag_version {
      publish_cookbook(options[:knife_config])
    }
  end

  private

    def clean?
      sh_with_excode("git diff --exit-code")[1] == 0
    end

    def current_version
      metadata = Chef::Cookbook::Metadata.new
      metadata.from_file(source_root.join("metadata.rb").to_s)
      metadata.version
    end

    def publish_cookbook(config)
      sh "knife cookbook site share artifact \"Utilities\" -o #{source_root.join("..")} -c #{config}"
    end

    def tag_version
      sh "git tag -a -m \"Version #{current_version}\" #{current_version}"
      say "Tagged: #{current_version}", :green
      yield if block_given?
      sh "git push --tags"
    rescue => e
      say "Untagging: #{current_version} due to error", :red
      sh_with_excode "git tag -d #{current_version}"
      say e, :red
      exit 1
    end

    def source_root
      Pathname.new File.dirname(File.expand_path(__FILE__))
    end

    def sh(cmd, dir = source_root, &block)
      out, code = sh_with_excode(cmd, dir, &block)
      code == 0 ? out : raise(out.empty? ? "Running `#{cmd}` failed. Run this command directly for more detailed output." : out)
    end

    def sh_with_excode(cmd, dir = source_root, &block)
      cmd << " 2>&1"
      outbuf = ''

      Dir.chdir(dir) {
        outbuf = `#{cmd}`
        if $? == 0
          block.call(outbuf) if block
        end
      }

      [ outbuf, $? ]
    end
end
