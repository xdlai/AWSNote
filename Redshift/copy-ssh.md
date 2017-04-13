# COPY data to Redshift via SSH

This document assumes you want Redshift to copy data from an ec2 instance via SSH.
And the data format between ec2 and Redshift is CSV.

## AWS permission requires

1. An IAM Role or AWS access key which allows the ec2 instance to access S3.
1. An IAM Role or AWS access key allows Redshift to access S3.

## Step by Step
1.  Get the public key of Redshift.
1.  Add the public key of Redshift into ec2-user's authorized_keys. authorized_keys is stored in
    */home/ec2-user/.ssh/authorized_keys*. 
1.  Get the public key of ec2. **Redshift only support RSA!** The RSA public is is stored at
    */etc/ssh/ssh_host_rsa_key.pub*. The content in the file will be like
    ```
    ssh-rsa AAAAB3NzaC1yc2....... root@ip-172-1-1-1
    ```
    All we need is "AAAAB3NzaC1yc2.......".
1.  Create manifest file for Redshift and put it to s3://<your bucket>/<path you want>
    ```
    {
        "entries": [
            {
                "endpoint":"<The host name of address of ec2 instance>",
                "command": "/bin/cat /home/ec2-user/ttttt.tbl",
                "mandatory":true,
                "username": "ec2-user",
                "publickey": "AAAAB3NzaC1yc2......."
            }
         ]
    }
    ```
    The example assumes the data csv file located at /home/ec2-user/ttttt.tbl.
    The public key here is the public key of ec2 instance.
    The command **/bin/cat /home/......** is just an example. You can replace it to any command
    you need. Just to make sure the command can generate csv for Redshift via standard output (STDOUT).
1.  Issue COPY query to Redshift.
    ```
    COPY <table name> 
    FROM 's3://<your bucket>/<path you want>'
    ACCESS_KEY_ID '<access key id>'
    SECRET_ACCESS_KEY '<secret access key>'
    ssh
    CSV;
    ```
    
## Something weired
I encountered "knownhost public key check failure". After reboot ec2, the issue is gone.... 
