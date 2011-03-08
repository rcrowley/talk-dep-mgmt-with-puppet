set :remote, "/home/rcrowley/work/pycon-2011"

role :www, "rcrowley.org"

task :static do
  system "showoff static"
end

task :deploy do
  static
  run "rm -rf #{remote}"
  upload "static", remote
  Dir["*.css", "*.js"].each do |filename|
    upload filename, "#{remote}/#{filename}"
  end
end
