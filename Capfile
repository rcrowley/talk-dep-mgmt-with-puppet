set :remote, "/home/rcrowley/work/carbon-five-2010-11-11"

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
