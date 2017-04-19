# Welcome to glFTPd best practice


So you want to run glFTPd.

DISCLAIMER: What you can read here is just my humble opinion.

Take everything with a grain of salt, keep having fun
and question everything.

You know it's all about fun. Even if it gets serious :-)

## Logging

Do not invent your own logging but use already established mechanisms for that.
One of them is syslog(). That's easy to use :-)

Recently the rsyslogd syslog server became popular.
You can use it on all the platforms that are supported by glFTPd.

Since glFTPd is run within chroot we provide a chrooted /dev/log file
and listen on localhost on some udp port to receive log messages

Then we format them and write that out to a logfile in ftp-data/logs
where software like pzs-ng can pick the messages up and process them
as events or something.

While this is an example for FreeBSD it also works with Leenox.
Just keep in mind that /var/run/log is /dev/log there.

    #   __  ____  __       ___
    #  / /_/ / /_/ /____  / _/
    # / __/ / __/ __/ _ \/ _/
    # \__/ /\__/\__/\___/_/
    #   /_/
    #  / rsyslog config
    module(load="imudp") # needs to be done just once
    input(type="imudp" inputname="imudp.glftpd" address="127.0.0.1" port="11514")
    module(load="imuxsock")
    input(type="imuxsock" HostName="chroot.glftpd" Socket="/opt/server/glftpd/var/run/log")
    $template glftpdFormat,"%timegenerated:::date-wdayname% %timegenerated:1:3% %timegenerated:::date-day% %timegenerated:::date-hour%:%timegenerated:::date-minute%:%timegenerated:::date-second% %timegenerated:::date-year% %syslogtag% %msg:2:$%\n"
    # Log anything from host glftpd to logfile
    if $hostname == 'chroot.glftpd' then /opt/server/glftpd/ftp-data/logs/syslog.log;glftpdFormat
    & stop
    if $inputname == 'imudp.glftpd' then /opt/server/glftpd/ftp-data/logs/syslog.log;glftpdFormat
    & stop

pzs-ng reads logfiles using regular expressions that differ for each known logtype.

    0. glftpdlog set regex ^.+ \d+:\d+:\d+ \d{4} (\S+): (.+)
    1. loginlog  see logtype 2
    2. sysoplog  set regex ^.+ \d+:\d+:\d+ \d{3} \[(\d+)\s*\] (.+)

so we use logtype glftpdlog in ngBot.conf:

    set glftpdlog(SYSLOG)       "$glroot/ftp-data/logs/syslog.log"

## cscripts and why scriptable does not necessarily mean you have to use a bash (coming up)

glFTPd is known as a scriptable ftp server. You can hook into most events that
occur during the session for example the creation of a directory or deletion
of a file and let a script decide if that action is allowed to take place or
not. Cool thing :-)

Just imagine a rat pack of clients rushing on your ftp server trying to create
directories. Your script gets executed very often within a short period
of time and that's when you realize your load increases quite a bit.

Soon users start to yell at you because they feel your ftp server lags like
hell.

Maybe that's because you used a shell script as a cscript which is perfectly
ok to do unless you could replace that script with a compiled program.

People tend to love golang or rust nowerdays but C is fine :-)

See [f-dirprecheck](http://www.high-society.at/f-scripts/#f-dirprecheck) 

My own goal is to not have any shell script within glroot/bin.

## configuration for the sitebot belongs solely into ngBot.conf (coming up)
Changes to pzs-ng happen from time to time. To also keep your sitebot
up to date use symlinks from the sitebot's installation path to the source
directory of pzs-ng which is regularly updated using git pull.

    ngBot.conf.defaults -> ../src/pzs-ng/sitebot/ngBot.conf.defaults
    ngBot.conf.dist -> ../src/pzs-ng/sitebot/ngBot.conf.dist
    ngBot.help -> ../src/pzs-ng/sitebot/ngBot.help
    ngBot.tcl -> ../src/pzs-ng/sitebot/ngBot.tcl
    ngBot.vars -> ../src/pzs-ng/sitebot/ngBot.vars

this implies that you absolutely have to avoid changes to these files!
All changes belong to ngBot.conf or you loos updateability.

Refrain from manually editing ngBot.tcl or modules/glftpd.tcl.
If you cannot live without editing those files better report it
to the devs on irc in #pzs-ng@efnet.

You can configure Plugins in ngBot.conf instead of editing the plugin's tcl
file like this

    variable ::ngBot::plugin::NickDb::libSQLite "/lib/sqlite3/libsqlite3.so"
    variable ::ngBot::plugin::Blow::keyx "false"
    set ::ngBot::plugin::Blow::blowkey(#winamp) "somepass"

## generate your configs to get tooled changes (coming up)
## version control is a thing (coming up)
## get your rights section straight (coming up)
## keep your libraries updated (coming up)
## uids and gids or why sitebot gets its own system user (coming up)
## encryption is mandatory (coming up)
