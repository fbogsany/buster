require 'shopify_api'

module Buster
  CYCLE_TIME = 0.5 # seconds

  def self.run(config)
    shop_url = "https://#{config['api_key']}:#{config['password']}@#{config['shop_name']}.myshopify.com/admin"
    ShopifyAPI::Base.site = shop_url
    trap(:INT, "EXIT")
    start_time = Time.now
    loop do
      products = ShopifyAPI::Product.all.select {|p| config['products'].include? p }
      products.each {|p| p.save }
      requests = products.count + 1
      throttle(start_time, requests)
      start_time = Time.now
    end
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
