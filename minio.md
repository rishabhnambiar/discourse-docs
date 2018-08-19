## Introducing Minio for Discourse

Minio is an object storage server released under Apache License v2.0.
It is compatible with Amazon S3 cloud storage service. It is best 
suited for storing unstructured data such as photos, videos, log files, 
backups and container/VM images. 

## Step 1: Configuring Minio

### Installing Minio Server

This guide assumes that you have already created a server in your preferred region on a VPS provider of your choice. We usually recommend [Digitalocean.](www.digitalocean.com) Store the IP address of the server for future use. (eg. 123.01.234.12).



We will be using the Minio Docker installer for simplicity, use the [Minio Quickstart Guide](https://docs.minio.io/docs/minio-quickstart-guide.html) to install from source or binaries.

Run `docker pull minio/minio` to download the latest stable Minio Docker image.

### Setup Authentication 

Create a Access Key ID/Secret Access Key pair of your choice and store it safely. Here's an example pair, please try to generate long, secure keys.

**Access Key ID**: EXAMPLEFODNN7EXAMPLE
**Secret Access Key**:  EXAMPLEKEYFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

Run the following command to start Minio Server using the keys you just created:
The command has been taken from the **Minio Custom Access and Secret Keys** section of the [Minio Quickstart Guide](https://docs.minio.io/docs/minio-quickstart-guide.html). 

```
docker run -p 9000:9000 --name minio1 \
  -e "MINIO_ACCESS_KEY=EXAMPLEFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=EXAMPLEKEYFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -v /mnt/data:/data \
  -v /mnt/config:/root/.minio \
  minio/minio server /data
```

If you wish to not generate your own keys, Run this command instead:
**Note:** Minio will auto-generate a new key pair every time you start the container. 

```
docker run -p 9000:9000 --name minio1 \
  -v /mnt/data:/data \
  -v /mnt/config:/root/.minio \
  minio/minio server /data
```

This is the output you should see after running the above command and starting the container:

```latest: Pulling from minio/minio
911c6d0c7995: Pull complete 
9952c099c0a8: Pull complete 
20127dd4dd25: Pull complete 
Digest: sha256:e8c43f24a6edb16a655553249a000aca24e176837c358d9e3e244dfac8b9c30c
Status: Downloaded newer image for minio/minio:latest

Created minio configuration file successfully at /root/.minio
Endpoint:  http://172.17.0.2:9000  http://127.0.0.1:9000
AccessKey: EXAMPLEFODNN7EXAMPLE 
SecretKey: EXAMPLEKEYFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

Browser Access:
   http://172.17.0.2:9000  http://127.0.0.1:9000

Command-line Access: https://docs.minio.io/docs/minio-client-quickstart-guide
   $ mc config host add myminio http://172.17.0.2:9000 EXAMPLEFODNN7EXAMPLE EXAMPLEKEYFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

These values will be required for logging into the Minio Browser on your IP address and using Minio Client.

## Step 2: Configuring Discourse for Minio

After setting up your Minio keys, the next step is to configure your Discourse instance. Make sure you're logged in with an administrator account and go the **Settings** section in the admin panel.

Type in "s3" in the textbox on the top-left to display only the relevant settings:

You will need to:

- Check the "`enable s3 backups`" checkbox if you want to activate manual or automated backups
  - Enter the desired Space name (bucket) in "`s3 backup bucket`" if `enable s3 backups` is checked
- Check the "`enable s3 uploads`" checkbox if you want to allow images to be uploaded and served by Minio
  - Enter the desired Space name (bucket) in "`s3 upload bucket`" if `enable s3 uploads` is checked
- Paste in both "`Access Key ID`" and "`Secret Access Key`" in their respective text fields
- In `s3 endpoint`, paste in the IP address of your VPS eg. `http://123.01.234.12`
- For `s3 region`, leave the field at it's default value. This is because your Minio Server instance is already located in the geographical location of your VPS and this setting is ignored.
- **NOTE:** Enable the `s3 force path style` checkbox for using Minio.

**What your settings should look like for Minio:**
_(Admin -> Settings -> Type 's3' in filter)_


					
**Note:** _You can enable only backups or only uploads or both._



## Step 3: Perform a Test Backup

Visit [/admin/backups](/admin/backups) on your Discourse instance when logged in as an Administrator. 
Click the `Backup` button to perform a private backup of your site.

To check if your backup was uploaded correctly, visit the Minio Browser app in your web browser.
**url:** http://<YOUR_SERVER_IP>:9000/

You should see a login page where you can enter your `Access Key ID` and `Secret Access Key`. You should see your uploaded backup file in the Minio Browser. If not, please check your credentials and url.

This test backup would create a bucket with the name you entered in site_settings.



## Step 4: Setting an upload Bucket Policy (skip if not using Minio for image uploads)

Till now, we have set up Minio for private files but if you want to use Minio for uploads, you have to make the files publicly accessible.  You will need to use the Minio Client app for configuring your bucket policies using a CLI. 

Install Minio Client from the [Client Quickstart Guide](https://docs.minio.io/docs/minio-client-quickstart-guide.html) on your **local development machine** on the OS of your choice. To test if `mc` (Minio Client) was installed, run `./mc --help` or `mc --help`.

Run: `mc config host add myminio http://<YOUR_SERVER_IP>:9000 EXAMPLEFODNN7EXAMPLE EXAMPLEKEYFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` 

- This command creates a Minio client called `myminio` for your server instance and IP address.

Before we set a policy for your upload bucket, you must first create your upload bucket using the Minio Browser app or the `mc` client. Run `mc mb myminio/<YOUR_UPLOAD_BUCKET_NAME>`.

Now run: `./mc policy download myminio/<YOUR_UPLOAD_BUCKET_NAME>` to set an upload policy for your complete bucket.
Expected output: `Access permission for myminio/<YOUR_UPLOAD_BUCKET_NAME> is set to download`.

## Enjoy
That's it. From now on, your images or backups will be uploaded to and served from Minio. 

---
#### Todo
- Configure SSL
- Configure domain??