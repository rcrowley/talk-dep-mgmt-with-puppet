set :remote, "/home/rcrowley/work/sfruby-2010-09-29"

role :www, "rcrowley.org"

task :static do
  system "showoff static"
end

task :deploy do
  static
  upload "static", remote, :via => :scp, :recursive => true
  Dir["*.css", "*.js"].each do |filename|
    upload filename, "#{remote}/#{filename}", :via => :scp
  end
end