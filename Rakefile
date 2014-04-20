require 'yaml'
require 'active_support/json/encoding'

desc "Build FpptyTips CDN json"
task :json do
  teams = Source.to_hash(:teams)
  Build.to_json({obj: teams, dest: :teams})

  fixture = Source.to_hash(:fixture)
  Build.to_json({obj: fixture, dest: :fixture})
end

class Source
  def self.to_hash(type)
    hash = YAML::load File.open("src/#{type.to_s}.yml")
  end
end

class Build
  def self.to_json(opts)
    File.open("build/#{opts[:dest].to_s}.json", 'w') do |f|
      f.puts ActiveSupport::JSON.encode(opts[:obj])
    end
  end
end