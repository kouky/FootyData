require 'yaml'
require 'active_support/json/encoding'
require 'active_support/core_ext/hash/indifferent_access'
require 'active_support/core_ext/object/blank'

desc "Build all json"
task :json => ['json:fixture', 'json:ladder'] do
end

namespace :json do

  desc "Build fixture json"
  task :fixture do
    fixture = Fixture.new()
    Build.to_json({obj: fixture.data, dest: :fixture})
  end

  desc "Build ladder json"
  task :ladder do
    ladder = Source.to_hash(:ladder)
    Build.to_json({obj: ladder, dest: :ladder})
  end

end

class Source
  def self.to_hash(type)
    YAML::load(File.open("src/#{type.to_s}.yml")).with_indifferent_access
  end
end

class Build
  def self.to_json(opts)
    File.open("build/#{opts[:dest].to_s}.json", 'w') do |f|
      f.puts ActiveSupport::JSON.encode(opts[:obj])
    end
  end
end

class Fixture
  attr_accessor :data

  def initialize
    @data = Source.to_hash(:fixture)
    @ladder = Source.to_hash(:ladder)
    enumerate_teams
  end

  private

  def enumerate_teams
    @data[:rounds].each do |round|
      round[:games].each do |game|
        game[:homeTeam] = find_team_by_short_name game[:homeTeam]
        game[:awayTeam] = find_team_by_short_name game[:awayTeam]
      end
    end
  end

  def find_team_by_short_name(short_name)
    team = @ladder[:teams].find do |team|
      team[:shortName] == short_name
    end

    raise "Could not find team #{short_name}" if team.blank?
    team
  end
end