require 'yaml'
require 'active_support/json/encoding'
require 'active_support/core_ext/hash/indifferent_access'
require 'active_support/core_ext/object/blank'
require 'fog'

desc "Build fixture and ladder json"
task :json do
  fixture = Fixture.new()
  Build.to_json({obj: fixture.data, dest: :fixture})

  ladder = Source.to_hash(:ladder)
  Build.to_json({obj: ladder, dest: :ladder})
end

desc "Sync json files to CDN"
task :sync do
  cloud = CloudFiles.new
  cloud.sync(:ladder)
  cloud.sync(:fixture)
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
    enumerate_game_ids
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

  def enumerate_game_ids
    @data[:rounds].each do |round|
      round[:games].each_with_index do |game, index|
        game[:id] = numeric_id_for_game(@data[:season], round[:id], index)
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

  def numeric_id_for_game(season, round_id, game_index)
    season_suffix = season.to_s[-2..-1]
    padded_round_id = "%02d" % round_id
    padded_game_id = "%02d" % (game_index + 1)
    (season_suffix + padded_round_id + padded_game_id).to_i
  end
end

class CloudFiles
  def initialize
    @rackspace = Rackspace.new()
    @storage = Fog::Storage.new({
      provider:           'Rackspace',
      rackspace_username: @rackspace.user_name,
      rackspace_api_key:  @rackspace.api_key,
      rackspace_region:   @rackspace.region.to_sym
    })
    @directory = @storage.directories.get @rackspace.container
  end

  def sync(type)
    file = @directory.files.create(
      key:    "#{type.to_s}.json",
      body:   File.open("build/#{type.to_s}.json"),
      public: true
    )
    file.save
  end
end

class Rackspace
  attr_reader :user_name, :api_key, :region, :container

  def initialize
    credentials = YAML::load(File.open(".rackspace.yml")).with_indifferent_access
    @user_name = credentials[:rackspace_username]
    @api_key = credentials[:rackspace_api_key]
    @region = credentials[:rackspace_region]
    @container = credentials[:rackspace_container]
  end
end