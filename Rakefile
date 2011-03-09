$: << File.expand_path(File.join(File.dirname(__FILE__), "lib"))

require 'set'
require 'fileutils'
require 'hpricot'
require 'date'
require 'fileutils'

DESTINATION = "build"

def globs(source)
  Dir.chdir(source) do
    dirs = Dir['*'].select { |x| File.directory?(x) }
    dirs -= ['_site']
    dirs = dirs.map { |x| "#{x}/**/*" }
    dirs += ['*']
  end
end

namespace :jekyll do
  task :initialize do
    require 'jekyll'
    @options = Jekyll.configuration('auto' => false, 'source' => 'site', 'destination' => DESTINATION)
    @site = Jekyll::Site.new(@options)
    @site.read_posts('')
  end
end

namespace :render do
  task :auto => ['render'] do
    require 'directory_watcher'

    puts "Auto-regenerating enabled: #{'site'} -> #{DESTINATION}"

    dw = DirectoryWatcher.new('site')
    dw.interval = 1
    dw.glob = globs('site')

    dw.add_observer do |*args|
      t = Time.now.strftime("%Y-%m-%d %H:%M:%S")
      puts "[#{t}] regeneration: #{args.size} files changed"
      Rake::Task['build:all'].invoke
      @site.process
    end

    dw.start

    loop { sleep 500 }
  end
end

task :render => 'jekyll:initialize' do
  FileUtils.rm_rf(DESTINATION)
  @site.process
end

task :publish => ['render'] do
  puts `rsync -ave ssh build/* tepper.pl:/var/www/tepper.pl`
  `curl "http://www.google.com/webmasters/tools/ping?sitemap=http%3A%2F%2Ftepper.pl%2Fsitemap.xml"`
end