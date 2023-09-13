# stomp-client-ruby


## Client class
The code for a ruby class polling the national rail stomp service could look similar to the code block below.

The `initialize` method sets up the client. For the `security_token` variable, use your security key from the 'My Feeds' page.

The `run` method will connect to the stomp service and subscribe to the queue. This process will run until the client is closed. When the client receives a new message from the queue, it will print the body of the message before sending an acknowledgment to the stomp service that the message was received.

The `shutdown` method can be called to close the connection and halt the process.

``` ruby
require "stomp"

class NrPoller

  def initialize
    @security_token = 'security-token-from-your-account-data-feeds-page'
 
    @username = 'd3user'
    @password = 'd3password'

    @hostname = 'datafeeds.nationalrail.co.uk'
    @port = '61613'
  end

  def run
    @client = Stomp::Client.new(@username, @password, @hostname, @port, true)
    puts @client.connection_frame.body

    @client.subscribe("/queue/#{@security_token}", { 'id' => @client.uuid(), 'ack' => 'auto' }) do |msg|
      puts msg.body
      @client.acknowledge(msg, msg.headers)
    end

    @client.join
    puts "Connected"
  end

  def shutdown
    if @client
      @client.close
    end
    puts "Disconnected"
  end

end

```

## Usage of the class

The script below will create an instance of the poller class described above and then call the `run` method, printing any new message from the queue to the console.

When `ctrl-c` is pressed, the `shutdown` method is called and the client disconnects cleanly.

``` ruby
nrPoller = NrPoller.new

begin
  nrPoller.run
rescue SystemExit, Interrupt
  nrPoller.shutdown
rescue Exception => e
  nrPoller.shutdown
  raise e
end
```
