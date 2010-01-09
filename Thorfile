class Spit < Thor
  desc 'start', 'run the SPIT generator'
  def start
    invoke :register
    invoke :call
    invoke :unregister
  end

  desc 'register', 'registers spit user with registrar'
  def register
    command = "#{config['sipp']} #{config['host']} -m 1 "
    command += [scenario_option('register'), user_option, authentication_option].join(' ')
    `#{command} 2> /dev/null`

    if $?.exitstatus.zero?
      puts "Registered #{config['user']}@#{config['domain']}"
    else
      puts "Registering #{config['user']}@#{config['domain']} failed"

      exit
    end
  end

  desc 'call', 'calls selected URIs'
  def call
    puts 'Calling...'
    command = "#{config['sipp']} #{config['host']} -m #{File.open('uris.csv').lines.count - 1} -l 2 -trace_shortmsg "
    command += [scenario_option('call'), user_option, inject_option].join(' ')
    `#{command} 2> /dev/null`

    puts 'Finished'
  end

  desc 'unregister', 'unregisters spit user with registrar'
  def unregister
    command = "#{config['sipp']} #{config['host']} -m 1 "
    command += [scenario_option('unregister'), user_option, authentication_option].join(' ')
    `#{command} 2> /dev/null`

    if $?.exitstatus.zero?
      puts "Unregistered #{config['user']}@#{config['domain']}"
    else
      puts "Unregistering #{config['user']}@#{config['domain']} failed"

      exit
    end
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

  def inject_option
    '-inf uris.csv'
  end
end
