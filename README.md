# rbarman

[![Build Status](https://travis-ci.org/sauspiel/rbarman.png?branch=master)](https://travis-ci.org/sauspiel/rbarman)

rbarman - Ruby Wrapper for 2ndQuadrant's PostgreSQL backup tool [barman](http://pgbarman.org)

## Installation

barman has to be installed and configured on the same host where this gem is to be used, otherwise it cannot get any useful information about your backups ;)


Add this line to your application's Gemfile:

    gem 'rbarman'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install rbarman

## Usage

Just a few examples. Please read the [documentation](https://rubygems.org/gems/rbarman) for more details.

### Get all your backups!

This will call various barman commands and some other sources (backup.info, xlog.db) to get information about your backups and could take several minutes (and  memory) if you have many backups with thousand of wal files. 

<pre>
servers = RBarman::Servers.all({:with_backups => true, :with_wal_files => true })
servers.count
=> 3

servers[0].name
=> "pgmaster"

servers[0].ssh_cmd
=> "ssh postgres@10.118.19.4"

servers[0].backups.count
=> 2

servers[0].backups.each { |b| p "id: #{b.id} }
=> "id: 20130304T080002"
=> "id: 20130225T192654"
=> "id: 20130218T080002"

backups = servers[0].backups
backups.latest.id
=> "20130304T080002"

backups.oldest.id
=> "20130218T080002"

backups[0].status
=> :done    # :started, :failed, :empty

backups[0].backup_start.to_s
=> "2013-03-04 08:00:02 +0100"

backups[0].backup_end.to_s
=> "2013-03-04 16:46:28 +0100"

backups[0].size
=> 201071500850 # bytes

backups[0].wal_file_size
=> 41875931136  # bytes

backups[0].timeline
=> 1

backups[0].begin_wal.xlog
=> "0000058F"

backups[0].end_wal.segment
=> "000000A5"

backups[0].wal_files.count
=> 9019

backups[0].wal_files[1022].compression
=> :bzip2
</pre>

### Get just one backup without wal files

<pre>
backup = RBarman::Backup.by_id('pg_master', '20130225T192654')
p "id: #{backup.id}|size: #{backup.size / (1024 ** 3) } GB|wal size: #{backup.wal_file_size / (1024 ** 3)} GB"
=> "id: 20130225T192654|size: 217GB|wal size: 72 GB"
</pre>

### Create a backup

Creates a new backup (and probably takes some time)

<pre>
b = RBarman::Backup.create('server')
p b.id
=> "20130304T131422"
</pre>

### Delete a backup

This instructs barman to delete the specified backup

<pre>
backup = RBarman::Backup.by_id('server', '20130225T192654', { :with_wal_files => false })
p backup.deleted
=> false
backup.delete
p backup.deleted
=> true
</pre>

### Recover a backup

Recovers newest/latest backup to the specified path on the remote host

<pre>
RBarman::Backups.all('testdb').latest.recover('/var/lib/postgresql/9.2/main', 
    { :remote_ssh_cmd => 'ssh postgres@10.20.20.2' })
</pre>

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request


## Meta

Written by [Holger Amann](http://github.com/hamann), sponsored by [Sauspiel GmbH](https://www.sauspiel.de)

Release under the [MIT License](http://www.opensource.org/licenses/mit-license.php)

