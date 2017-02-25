#Backup of the DB
#####(25/02/2017)

I needed to backup all the database data for one of my current projects and thought to create a task for that would be a good idea. I do like to have things as simpler as possible, so didn't want any gem or third party software in charge of this task. It's quite simple

```
namespace :db do  desc "Backup database to AWS-S3"
  task :backup => [:environment] do
    root_folder = '/tmp/'
    s3_folder = 'db_backups'
    backup_filename = "#{Time.now.strftime('%Y%m%d%H%M%S')}_backup_db"

    `pg_dump mydatabase > #{root_folder}#{backup_filename}`
    `gzip -9 #{root_folder}#{backup_filename}`

    bucket_name = ENV['S3_BUCKET'] 
    Aws.config.update({
      region: 'eu-west-1',
      credentials: Aws::Credentials.new(ENV['AWS_ACCESS_KEY_ID'], ENV['AWS_SECRET_ACCESS_KEY'])
    })
    s3 = Aws::S3::Resource.new

    s3.bucket(bucket_name).object("#{s3_folder}/#{backup_filename}.gz").upload_file("#{root_folder}#{backup_filename}.gz")
    `rm -f #{root_folder}#{backup_filename}.gz`
  end
end
```

For get this script working, the current user should have the proper rights to access to the database schema "mydatabase" and some environment variables created, such as S3_BUCKET, AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

This will upload the compressed dump file to the folder "db_backups" within the bucket configured in S3_BUCKET.

Then, you just need to create a cron to run the task whenever you want:

```
crontab -e
```

Add the following line
```
00 15,23 * * * /bin/bash -l -c 'THE_ROOT_TO_THE_BIN/rake -f THE_ROOT_TO_THE_RAKEFILE/Rakefile db:backup' >> LOGS_FOLDER/crontab.log
```

This will create a backup of your database twice a day, at 3pm and 23pm, and upload them to your S3 bucket