require 'rake'
require 'erb'

dist_name = :ubuntu
if File.exists?('/etc/arch-release')
  dist_name = :archlinux
end

desc "test some thing."
task :test do
  puts "Linux distribution: #{dist_name.to_s}"
end

desc "install the dot files into user's home directory"
task :install do
  home_dir = ENV['HOME']
  current_dir = File.dirname(__FILE__)
  if not home_dir
    puts "Fatal error: no home_dir path given.\nPlease make sure that HOME is set in your environment.\nAborting."
    exit 1
  end

  $replace_all = false

  # setup home dir's dot file.
  Dir.chdir('homefile') do
    Dir['*'].each do |file|
      next if %w[Rakefile README.rdoc LICENSE].include? file


      # not clean file
      if %w[bin].include? file
        install_file(file, File.join(home_dir, file))
      # Install files in "config" separately
      elsif 'config' == file
        Dir[File.join(file, '*')].each do |file|
          clean_name = file.sub(/\.erb$/, '')
          install_file(file, File.join(home_dir, "."+clean_name))
        end
      else
        clean_name = file.sub(/\.erb$/, '')
        install_file(file, File.join(home_dir, "."+clean_name))
      end
    end
  end

  # update Xresources
  system "xrdb -load ~/.Xresources"

  # copy /usr/bin file
  system "sudo cp #{current_dir}/package/usr/bin/* /usr/bin -r"
end


def install_file(file, target)
  nice_target = target.sub(/#{ENV['HOME']}/, '~') # for display: collapse "~"
  if File.exist?(target)
    if File.identical? file, target
      puts "identical #{nice_target}"
    elsif $replace_all
      replace_file(file, target)
    else
      print "overwrite #{nice_target}? [ynaq] "
      case $stdin.gets.chomp
      when 'a'
        $replace_all = true
        replace_file(file, target)
      when 'y'
        replace_file(file, target)
      when 'q'
        exit
      else
        puts "skipping #{nice_target}"
      end
    end
  else
    link_file(file, target)
  end
end

def replace_file(file, target)
  system %Q{rm -rf "#{target}"}
  link_file(file, target)
end

def link_file(file, target)
  nice_target = target.sub(/#{ENV['HOME']}/, '~') # for display: collapse "~"
  if file =~ /.erb$/
    puts "generating #{nice_target}"
    File.open(target, 'w') do |new_file|
      new_file.write ERB.new(File.read(file)).result(binding)
    end
  else
    puts "linking #{nice_target}"
    system %Q{ln -sfn "$PWD/#{file}" "#{target}"}
  end
end
