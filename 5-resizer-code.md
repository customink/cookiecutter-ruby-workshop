[◀️ PREV: Image Processing & Lambda Layers](4-image-processing.md)

# Step 5: Resizer Code & Permissions

In this section we are going to respond to our new S3 events from the [Connecting Lambda to S3](3-connecting-lambda-to-s3.md) section. Here are the steps our code will perform:

- Read the image file from our S3 bucket.
- Resize the image to 200px wide as a new image.
- Upload that image back to the same S3 bucket with a new name.


## Resizer Class

I wrote this plain old ruby object (PORO) class to do the work for us. Please take some time to read over it. Here are some high level call outs:

- Require the ImageProcessing gem's [Vips](https://github.com/janko/image_processing/blob/master/doc/vips.md) implementation.
- Creates an `Aws::S3::Client` client to use.
- Initializes with an S3 key and optional desired size.
- Resizes the image if it is not resized already.
- Leverage S3 metadata to avoid recursive resizing events.
- Does all work in memory using buffers vs writing to disk.

Create a new file in your project within the `lib/image_service` directory named `resizer.rb` file using the below code. 💥**Please change `SOMEUNIQNAME` for the `BUCKET` constant to match your actual bucket name.** 💥

⚡️⏩⚡️

```ruby
require 'aws-sdk-s3'
require 'image_processing/vips'

module ImageService
  class Resizer
    CLIENT = Aws::S3::Client.new region: Env.region
    BUCKET = "image-service-SOMEUNIQNAME-#{Env.stage}"

    attr_reader :key, :size

    def initialize(key, size = 200)
      @key = key
      @size = size
    end

    def resize!
      return if resized?
      resized = ImageProcessing::Vips
        .source(source)
        .resize_to_limit(size, nil)
        .call(save: false)
        .write_to_buffer(File.extname(key))
      put_object resized, new_key
    end

    def source
      if File.extname(key) == '.png'
        Vips::Image.pngload_buffer(get_object)
      else
        Vips::Image.new_from_buffer(get_object, File.extname(key))
      end
    end

    def new_key
      "#{File.basename(key, File.extname(key))}-#{size}#{File.extname(key)}"
    end

    def get_object
      CLIENT.get_object(bucket: BUCKET, key: key).body.read
    end

    def put_object(object, put_key)
      CLIENT.put_object bucket: BUCKET, key: put_key, body: object,
                        metadata: { 'resized' => '1' }
    end

    def resized?
      metadata['resized'] == '1'
    end

    def metadata
      response = CLIENT.head_object bucket: BUCKET, key: key
      response.metadata || {}
    rescue Aws::S3::Errors::NotFound
      {}
    end

  end
end
```

❓I noticed you did a require for `aws-sdk-s3` but it was not added to your `Gemfile`, how does that work? Each Lambda runtime has preinstalled SDK packages to make common tasks easier. In this case the [Aws::S3](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Client.html) SDK is one of them.

❓I noticed you did not pass an `AWS_ACCESS_KEY_ID` nor a `AWS_SECRET_ACCESS_KEY` to the client? How does this work? Since our Lambda is invoked with an execution role, added in our [previous step](4-image-processing.md), we do not have to manage keys or secrets. All client SDKs will use that role automatically. Welcome to the future!

## Handler Changes

Now that we have our little Ruby object to resize S3 objects we need to hook it up in our Lambda's handler function. The code below has been changed to:

- Added a require for our new file.
- Get the S3 key form the event.
- Create a new Resizer instance. Resize the file.
- Return some meaningful response. The S3 key.

Update your `image_service.rb` file to use this code.

⚡️⏩⚡️

```ruby
require 'json'
require 'dotenv'
require_relative './image_service/env'
require_relative './image_service/resizer'

def handler(event:, context:)
  key = event['Records'][0].dig('s3','object','key')
  resizer = ImageService::Resizer.new(key, 200)
  resizer.resize!
  { statusCode: 200,
    headers: [{'Content-Type' => 'application/json'}],
    body: JSON.dump({key: key}) }
end
```

## Deploy Updated Stack

That was a lot to change but we are now ready to deploy and see our efforts at work.

⚡️⏩⚡️

```shell
$ STAGE_ENV=production ./bin/deploy
```

## See It At Work

<a href="RubyForGood.png"><img alt="Ruby For Good S3 Test Image" align="right" width="250" src="https://user-images.githubusercontent.com/2381/80309783-159a2a80-87a5-11ea-9c73-d036b683d245.png"></a>Just like in the [Connecting Lambda to S3](3-connecting-lambda-to-s3.md) step we are going to upload our test image, `RubyForGood.png` as seen to the right. 

- Click on "Services" in the toolbar.
- Enter "S3" into the find services field, select.
- Click on your S3 bucket named `image-service-SOMEUNIQNAME-production`. Where SOMEUNIQNAME is the name you chose earlier.
- Delete the file we uploaded before.

Now drag and drop our `RubyForGood.png` file into the S3 browser window and lick "Next" 3 times to use the default values for permissions, etc. Finally click "Upload" to start the upload. After it uploads hit the ♻️ button a few times till you see the new file show up.

<img width="853" alt="New RubyForGood.png Resized" src="https://user-images.githubusercontent.com/2381/80836221-4c67aa80-8bc2-11ea-9e2a-0e7e04480891.png">

💥 Did you not see the new file? Go check your CloudWatch logs to see what happened. If you have any issues or get stuck, reach out for help using the [GitHub Issues](https://github.com/customink/cookiecutter-ruby-workshop/issues) and I would be happy to help.

## 🚀 CI/CD

If everything worked out, now is a good time to commit your code.

<br/>
<br/>

[▶️ NEXT: Next Steps & Resources](6-resources.md)

<br/>
<br/>
