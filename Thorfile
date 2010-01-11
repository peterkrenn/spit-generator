class Spit < Thor
  desc 'start', 'run the SPIT generator'
  method_options :ip => :required
  method_options :transport => :string
  method_options :'uri-file' => :string
  def start
    invoke :register
    invoke :call
    invoke :unregister
    invoke :generate_report
  end

  desc 'register', 'registers spit user with registrar'
  method_options :ip => :required
  method_options :transport => :string
  def register
    command = "#{config['sipp']} #{config['host']} -m 1 -i #{options['ip']} "
    command += [scenario_option('register'),
                user_option,
                authentication_option,
                transport_option(options)].join(' ')
    `#{command} 2> /dev/null`

    if $?.exitstatus.zero?
      puts "Registered #{config['user']}@#{config['domain']}"
    else
      puts "Registering #{config['user']}@#{config['domain']} failed"

      exit
    end
  end

  desc 'call', 'calls selected URIs'
  method_options :ip => :required
  method_options :transport => :string
  method_options :'uri-file' => :string
  def call
    puts 'Calling...'

    uri_file = options['uri-file'] || config['uri_file']

    command = "#{config['sipp']} #{config['host']} -m #{File.open(uri_file).lines.count - 1} -l 2 -trace_shortmsg -i #{options['ip']} "
    command += [scenario_option('call'),
                user_option,
                inject_option(uri_file),
                transport_option(options)].join(' ')
    `#{command} 2> /dev/null`

    puts 'Finished'
  end

  desc 'unregister', 'unregisters spit user with registrar'
  method_options :ip => :required
  method_options :transport => :string
  def unregister
    command = "#{config['sipp']} #{config['host']} -m 1 -i #{options[:ip]} "
    command += [scenario_option('unregister'),
                user_option,
                authentication_option,
                transport_option(options)].join(' ')
    `#{command} 2> /dev/null`

    if $?.exitstatus.zero?
      puts "Unregistered #{config['user']}@#{config['domain']}"
    else
      puts "Unregistering #{config['user']}@#{config['domain']} failed"

      exit
    end
  end
  
  desc 'generate_report', 'displays report from pending calls and exports it to logs'
  def generate_report
    parse_log_files
    export_calls
    print_log

    invoke :clear_logs
  end

  desc 'clear_logs', 'deletes all log files'
  def clear_logs
    FileUtils.rm(Dir['scenarios/*.log'])
  end

  private

  def config
    unless @config
      require 'yaml'

      @config = YAML.load_file 'config.yml'
    end

    @config
  end

  def scenario_option(name)
    "-sf scenarios/#{name}.xml"
  end

  def user_option
    "-key user #{config['user']} -key domain #{config['domain']}"
  end

  def authentication_option
    "-s #{config['user']} -ap #{config['password']}"
  end

  def inject_option(uri_file)
    "-inf #{uri_file}"
  end

  def transport_option(options)
    '-t t1' if config['transport'] == 'tcp' or options['transport'] == 'tcp'
  end

  def parse_log_files
    @calls = []

    require 'csv'

    entries = []
    files = Dir['scenarios/*.log']

    return if files.empty?

    CSV.open(files.last, 'r', "\t") do |row|
      entries << row
    end

    entries.group_by { |entry| entry[4] }.each do |key, group|
      call = {}
      call[:pid] = group[0][4].to_s[/\d+-(\d+)/, 1].to_i
      call[:id]= group[0][4].to_s[/^\d+/].to_i
      call[:uri] = group[0][6].to_s[/sip:(\S+)/, 1]

      if group[1][6] =~ /404 Not Found/
        call[:status] = 'is not registered'
      elsif group[2][6] =~ /180 Ringing/
        group[3..-1].each do |entry|
          if entry[6] =~ /200 OK/
            call[:picked_up_at] = entry[2].to_f
            call[:picked_up_after] = entry[2].to_f - group[2][2].to_f
            break
          end
        end

        unless call[:picked_up_at]
          call[:status] = 'did not pick up'
        else
          call[:status] = 'picked up'
          group[4..-1].each do |entry|
            if entry[6] =~ /BYE/
              call[:duration] = entry[2].to_f - call[:picked_up_at]
              break
            end
          end
        end
      else
        call[:status] = 'has an unknown status'
      end

      @calls << call
    end
  end

  def export_calls
    require 'csv'

    CSV.open("logs/#{Time.now.strftime('%Y-%m-%d')}-#{@calls.first[:pid]}-calls.log", 'w') do |file|
      @calls.each do |call|
        file << [call[:pid], call[:id], call[:uri], call[:status], call[:picked_up_after], call[:duration]]
      end
    end unless @calls.empty?
  end

  def print_log
    @calls.each do |call|
      case call[:status]
        when 'picked up':
          puts "#{call[:uri]} picked up after #{call[:picked_up_after].round} seconds and was on the line for #{call[:duration].round} seconds"
        else
          puts "#{call[:uri]} #{call[:status]}"
      end
    end
  end
end
