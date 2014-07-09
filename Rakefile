require 'shopify_api'

module Buster
  CYCLE_TIME = 0.5 # seconds

  def self.run(config)
    shop_url = "https://#{config['api_key']}:#{config['password']}@#{config['shop_name']}.myshopify.com/admin"
    ShopifyAPI::Base.site = shop_url
    last_time = start_time = Time.now
    updates = 0
    trap(:INT) { print "\n"; exit }
    loop do
      products = ShopifyAPI::Product.all.select {|p| config['products'].include? p.handle }
      products.each {|p| p.save }
      requests = products.count + 1
      updates += products.count
      throttle(last_time, requests)
      progress(start_time, updates)
      last_time = Time.now
    end
  end

  def self.terminal_width
    `stty size`.scan(/\d+/).map { |s| s.to_i }[1]
  end

  def self.progress(start_time, updates)
    print "\r"
    print(" " * (terminal_width - 1))
    print "\r#{updates} updates in #{(Time.now - start_time).to_i} sec"
  end

  def self.throttle(start_time, requests)
    stop_time = Time.now
    processing_duration = stop_time - start_time
    wait_time = (CYCLE_TIME * requests - processing_duration).ceil
    sleep wait_time if wait_time > 0
  end
end

desc "Run buster indefinitely"
task :buster do
  config_file = File.join(Rake.original_dir, ENV['BUSTER_CONFIG'])
  abort "BUSTER_CONFIG does not refer to a valid file: #{config_file}" unless File.exist?(config_file)
  Buster.run(YAML.load(File.read(config_file)))
end

task :default => :buster
