---
layout: post
title:  "Querying Chef REST Api With pure Ruby"
date:   2016-01-19 05:35:48
categories: chef ruby rest httparty raw api
---
If you want to talk to the chef API without using some wrapper like Ridley, knife, etc.., this is how you do it:

Note: This is taken from [here](https://gist.github.com/gmcmillan/3184964) but modified to use httparty
{% highlight ruby %}
require 'base64'
require 'time'
require 'digest/sha1'
require 'openssl'
require 'net/https'
require 'json'
require 'pry'
require 'httparty'

class ChefAPI
  # Public: Gets/Sets the String path for the HTTP request.
  attr_accessor :path

  # Public: Gets/Sets the String client_name containing the Chef client name.
  attr_accessor :client_name

  # Public: Gets/Sets the String key_file that is path to the Chef client PEM file.
  attr_accessor :key_file

  # Public: Initialize a Chef API call.
  #
  # opts - A Hash containing the settings desired for the HTTP session and auth.
  #        :server       - The String server that is the Chef server name (required).
  #        :port         - The String port for the Chef server (default: 443).
  #        :use_ssl      - The Boolean use_ssl to use Net::HTTP SSL
  #                        functionality or not (default: true).
  #        :ssl_insecure - The Boolean ssl_insecure to skip strict SSL cert
  #                        checking (default: OpenSSL::SSL::VERIFY_PEER).
  #        :client_name  - The String client_name that is the name of the Chef
  #                        client (required).
  #        :key_file     - The String key_file that is the path to the Chef client
  #                        PEM file (required).
  def initialize(opts = {})
    @server            = opts[:server]
    port              = opts.fetch(:port, 443)
    use_ssl           = opts.fetch(:use_ssl, true)
    ssl_insecure      = opts[:ssl_insecure] ? OpenSSL::SSL::VERIFY_NONE : OpenSSL::SSL::VERIFY_PEER
    @client_name      = opts[:client_name]
    @key_file         = opts[:key_file]
  end

  # Public: Make the actual GET request to the Chef server.
  #
  # req_path - A String containing the server path you want to send with your
  #            GET request (required).
  #
  # Examples
  #
  #   get_request('/environments/_default/nodes')
  #   # => ["server1.com","server2.com","server3.com"]
  #
  # Returns different Object type depending on request.
  def get_request(req_path, body)
    @path = req_path
    reqpath = @server + req_path
    begin
      response = HTTParty.get(reqpath, headers: headers(body, 'GET'))
      response.parsed_response
      # JSON.parse(response.body).keys
    rescue OpenSSL::SSL::SSLError => e
      raise "SSL error: #{e.message}."
    end
  end

  def put_request(req_path, body)
    @path = req_path
    reqpath = @server + req_path
    puts reqpath
    begin
      response = HTTParty.put(reqpath, headers: headers(body, 'PUT'), body: body)
      response.parsed_response
    rescue OpenSSL::SSL::SSLError => e
      raise "SSL error: #{e.message}."
    end
  end

  def post_request(req_path, body)
    @path = req_path
    reqpath = @server + req_path
    puts reqpath
    begin
      puts headers(body, 'POST')
      response = HTTParty.post(reqpath, headers: headers(body, 'POST'), body: body)
      response.parsed_response
    rescue OpenSSL::SSL::SSLError => e
      raise "SSL error: #{e.message}."
    end
  end

  def delete_request(req_path, body)
    @path = req_path
    reqpath = @server + req_path
    puts reqpath
    begin
      response = HTTParty.delete(reqpath, headers: headers(body, 'DELETE'), body: body)
      response.parsed_response
    rescue OpenSSL::SSL::SSLError => e
      raise "SSL error: #{e.message}."
    end
    end

  private

  # Private: Encode a String with SHA1.digest and then Base64.encode64 it.
  #
  # string - The String you want to encode.
  #
  # Examples
  #
  #   encode('hello')
  #   # => "qvTGHdzF6KLavt4PO0gs2a6pQ00="
  #
  # Returns the hashed String.
  def encode(string)
    ::Base64.encode64(Digest::SHA1.digest(string)).chomp
  end

  # Private: Forms the HTTP headers required to authenticate and query data
  # via Chef's REST API.
  #
  # Examples
  #
  #   headers
  #   # => {
  #     "Accept"                => "application/json",
  #     "X-Ops-Sign"            => "version=1.0",
  #     "X-Ops-Userid"          => "client-name",
  #     "X-Ops-Timestamp"       => "2012-07-27T20:09:25Z",
  #     "X-Ops-Content-Hash"    => "JJKXjxksmsKXM=",
  #     "X-Ops-Authorization-1" => "JFKXjkmdkDMKCMDKd+",
  #     "X-Ops-Authorization-2" => "JFJXjxjJXXJ/FFjxjd",
  #     "X-Ops-Authorization-3" => "FFJfXffffhhJjxFJff",
  #     "X-Ops-Authorization-4" => "Fjxaaj2drg5wcZ8I7U",
  #     "X-Ops-Authorization-5" => "ffjXeiiiaHskkflllA",
  #     "X-Ops-Authorization-6" => "FjxJfjkskqkfjghAjQ=="
  #   }
  #
  # Returns a Hash with the necessary headers.
  def headers(body, method)
    body      = body
    timestamp = Time.now.utc.iso8601
    key       = OpenSSL::PKey::RSA.new(File.read(key_file))
    canonical = "Method:#{method}\nHashed Path:#{encode(path)}\nX-Ops-Content-Hash:#{encode(body)}\nX-Ops-Timestamp:#{timestamp}\nX-Ops-UserId:#{client_name}"

    header_hash = {
      'Accept'             => 'application/json',
      'X-Ops-Sign'         => 'version=1.0',
      'X-Chef-Version'     => '12.3.1',
      'X-Ops-Userid'       => client_name,
      'X-Ops-Timestamp'    => timestamp,
      'X-Ops-Content-Hash' => encode(body),
      'Content-Type' => 'application/json'
    }

    signature = Base64.encode64(key.private_encrypt(canonical))
    signature_lines = signature.split(/\n/)
    signature_lines.each_index do |idx|
      key = "X-Ops-Authorization-#{idx + 1}"
      header_hash[key] = signature_lines[idx]
    end

    header_hash
  end
end
# chef_url = 'https://<ip>/organizations/default'
chef_url = 'https://<ip>'
chef = ChefAPI.new(server: chef_url, port: 443, ssl_insecure: true, client_name: 'pivotal', key_file: '/etc/opscode/pivotal.pem')

puts chef.delete_request('/organizations/test1', '')
# puts chef.delete_request('/organizations/secure', "")

# puts chef.get_request('/organizations/default/cookbooks', "")
# puts chef.get_request('/organizations/test1/nodes', "")

test = { 'name' => 'test1', 'full_name' => 'major test1' }
resp =  chef.post_request('/organizations', test.to_json)
puts resp
File.open("#{resp['clientname']}.pem", 'w') { |file| file.write(resp['private_key']) }

puts 'Users in org: '
puts chef.get_request('/organizations/test1/users', '')
puts chef.post_request('/organizations/test1/association_requests', { 'user' => 'admin' }.to_json)
resp =  chef.get_request('/organizations/test1/association_requests', '')
id = resp.find { |x| x['username'] == 'admin' }['id']
puts chef.put_request("/users/admin/association_requests/#{id}", { 'response' => 'accept' }.to_json)
puts 'Users in org: '
puts chef.get_request('/organizations/test1/users', '')

# puts chef.post_request('/organizations', {"name" => "secure", "full_name" => 'secure test1'}.to_json)
{% endhighlight %}


