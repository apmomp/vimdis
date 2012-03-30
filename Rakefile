require 'net/https'
require 'uri'

def download_file_through_http(url, file, recursion_limit = 5)
  raise ArgumentError, 'HTTP redirect too deep' if recursion_limit == 0

  uri = URI.parse(url)

  if uri.scheme == 'http'
    http = Net::HTTP.new(uri.host)
  elsif uri.scheme == 'https'
    http = Net::HTTP.new(uri.host, 443)
    http.use_ssl = true
  end

  http.start do
    response = http.get(uri.path)

    case response
    when Net::HTTPSuccess
      File.open(file, 'w') do |file|
        file.write(response.body)
      end
    when Net::HTTPRedirection
      download_file_through_http(response['location'], file, recursion_limit - 1)
    else
      response.error!
    end
  end
end

def dependency_satisfied?(dep)
  if system("which #{dep} > /dev/null 2>&1") && $?.exitstatus == 0
    true
  else
    false
  end
end

def extract_name(repo)
  match = repo.match(/^(git|http|https):\/\/.*\/(.*).git$/)

  if match.nil?
    puts "\n#{repo} doesn't seem like a valid git repo."

    nil
  else
    match[2]
  end
end

def install_bundle(repo, extra_options = {}, &block)
  name = extract_name(repo)
  dest = File.expand_path("../bundle/#{name}", __FILE__)

  if File.exists?(dest)
    puts "#{name} already installed"
  else
    puts "\nInstalling #{name}"

    if extra_options[:depends_on].is_a?(String)
      if !dependency_satisfied?(extra_options[:depends_on])
        puts "Dependency not met"

        return
      end
    elsif extra_options[:depends_on].is_a?(Array)
      extra_options[:depends_on].each do |dependency|
        if !dependency_satisfied?(extra_options[:depends_on])
          puts "Dependencies not met"

          return
        end
      end
    end

    system("git clone #{repo} #{dest}")

    if block
      Dir.chdir(dest) do
        block.call
      end
    end
  end
end

namespace :bundle do
  desc 'Install basic utils'
  task :basic do
    install_bundle('https://github.com/vim-scripts/molokai.git')
    install_bundle('https://github.com/mileszs/ack.vim.git', :depends_on => 'ack')
    install_bundle('https://github.com/wincent/Command-T.git') do
      if dependency_satisfied?('rvm')
        system('rvm system do rake make')
      else
        system('rake make')
      end
    end
    install_bundle('https://github.com/scrooloose/nerdcommenter.git')
    install_bundle('https://github.com/scrooloose/nerdtree.git')
    install_bundle('https://github.com/honza/snipmate-snippets.git')
    install_bundle('https://github.com/tomtom/tlib_vim.git')
    install_bundle('https://github.com/MarcWeber/vim-addon-mw-utils.git')
    install_bundle('https://github.com/tpope/vim-fugitive.git')
    install_bundle('https://github.com/tpope/vim-git.git')
    install_bundle('https://github.com/kana/vim-smartinput.git')
    install_bundle('https://github.com/garbas/vim-snipmate.git')
    install_bundle('https://github.com/tpope/vim-surround.git')
  end

  desc 'Install JavaScript utils'
  task :javascript do
    install_bundle('https://github.com/wookiehangover/jshint.vim.git', :depends_on => 'node')
    install_bundle('https://github.com/pangloss/vim-javascript.git')
  end

  desc 'Install Rails utils'
  task :rails do
    install_bundle('https://github.com/tpope/vim-rails.git')
    install_bundle('https://github.com/vim-ruby/vim-ruby.git')
  end
end

desc 'Install basic system'
task :install => ['setup_environment', 'bundle:basic'] do

end

desc 'Setup environment'
task :setup_environment => ['setup_environment:backup', 'setup_environment:link_config_files', 'setup_environment:create_dirs', 'setup_environment:install_pathogen'] do

end

namespace :setup_environment do
  desc 'Backup old config files'
  task :backup do
    puts "\nBacking up old config files"

    %w(.vimrc .gvimrc).each do |file|
      src = File.expand_path("~/#{file}")
      dest = File.expand_path("~/#{file}.backup")

      if File.exist?(src)
        puts("Moving #{src} to #{dest}")

        File.rename(src, dest)
      end
    end
  end

  desc 'Create dirs'
  task :create_dirs do
    %w(autoload backup bundle swap).each do |file|
      Dir.mkdir(file) unless File.exists?(file)
    end
  end

  desc 'Install Pathogen'
  task :install_pathogen do
    puts "\nInstalling pathogen.vim"
    download_file_through_http('https://github.com/tpope/vim-pathogen/raw/master/autoload/pathogen.vim', 'autoload/pathogen.vim')
  end

  desc 'Link config files'
  task :link_config_files do
    puts "\nLinking config files"

    %w(vimrc gvimrc).each do |file|
      dest = File.expand_path("~/.#{file}")

      unless File.exist?(dest)
        FileUtils.ln_s(File.expand_path("../#{file}", __FILE__), dest)
      end
    end
  end
end

desc 'Update vimdis'
task :update do
  puts "\nUpgrading vimdis:"
  system('git pull')
end

desc 'Upgrade vimdis'
task :upgrade => ['upgrade:cleanup', 'install'] do

end

namespace :upgrade do
  desc 'Cleanup repo'
  task :cleanup do
    system('git clean -dxf')
  end
end
