---
layout: post
title:  "Using Amazon S3 with Django and easy-thumbnails"
---

I recently worked on a project that asked users to submit images for a contest. 
I needed a way to create a gallery of these images after they were submitted and planned
to use the Django [easy-thumbnail](http://easy-thumbnails.readthedocs.org/) app to 
crop the uploaded images for thumbnails.

The website was going to be hosted with Heroku and I wanted to use Amazon S3 to
serve the user's submitted photos. I had used the easy thumbnails application on
sites serving local media in the past, but I had never used it with S3. 
Luckily getting S3 to work with Django is pretty straightforward thanks to 
the [django-storages app](http://django-storages.readthedocs.org).


Getting Django, S3, and easy_thumbnails to work together:
---
1. Go to [http://aws.amazon.com](http://aws.amazon.com/) and create an AWS account. 
   Then use the AWS console to create an S3 bucket to store your files.
2. Install the package requirements.
    - `pip install django-storages`
    - `pip install boto`  (*django-storages' docs recommended it*)
    - `pip install django-extensions` (*for syncing the existing MEDIA_ROOT directory with your S3 bucket*)
    - `pip install easy-thumbnails`
3. Add the required apps to your django INSTALLED_APPS setting:
    - `('easy_thumbnails', 'storages’, django_extensions')`
4. Run `manage.py syncdb` or `manage.py migrate` to install easy_thumbnails's database tables
5. Add the following settings in your Django settings.py file:

        {% highlight python %}
            # FOR EASY THUMBNAILS
            THUMBNAIL_SUBDIR = "_thumbs" # subdirectory name to store thumbs
            THUMBNAIL_PREFIX = "thumbs_" # filename prefix for thumbnails
            THUMBNAIL_DEFAULT_STORAGE = 'storages.backends.s3boto.S3BotoStorage' #storage backend for thumbnails


            # FOR DJANGO STORAGES
            DEFAULT_FILE_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
            AWS_ACCESS_KEY_ID = <YOUR ACCESS KEY> # get this key from your amazon account<
            AWS_SECRET_ACCESS_KEY = <YOUR SECRET KEY> # these can be accessed from your AWS admin
            AWS_STORAGE_BUCKET_NAME = <NAME OF YOUR BUCKET>

            # FOR COMMAND EXTENSIONS it uses a different setting. Docs say you can pass this on the command line but it did not work when I tried it
            AWS_BUCKET_NAME = AWS_STORAGE_BUCKET_NAME
        {% endhighlight %}

<ol start="6">
<li>Sync your static and uploaded media with S3
    - `./manage.py sync_media_s3 YOUR_BUCKET_NAME`  
     *This command will upload any files under MEDIA_ROOT to your s3 account*
    - `./manage.py collectstatic`  
    *This command will upload static media to your S3 account*
</li>
</ol>

Now any files you upload with Django will be stored on Amazon S3, and
easy_thumbnails will use S3 to store the thumbnails it creates from your
uploaded images.
