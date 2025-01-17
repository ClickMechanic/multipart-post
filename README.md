# Multipart::Post

Adds a streamy multipart form post capability to `Net::HTTP`. Also supports other
methods besides `POST`.

[![Ruby CI](https://github.com/socketry/multipart-post/actions/workflows/ruby.yml/badge.svg)](https://github.com/socketry/multipart-post/actions/workflows/ruby.yml)

## Features/Problems

* Appears to actually work. A good feature to have.
* Encapsulates posting of file/binary parts and name/value parameter parts, similar to
  most browsers' file upload forms.
* Provides an `UploadIO` helper class to prepare IO objects for inclusion in the params
  hash of the multipart post object.

## Installation

    gem install multipart-post

or in your Gemfile

    gem 'multipart-post'

## Usage

```ruby
require 'net/http/post/multipart'

url = URI.parse('http://www.example.com/upload')
File.open("./image.jpg") do |jpg|
  req = Net::HTTP::Post::Multipart.new url.path,
    "file" => UploadIO.new(jpg, "image/jpeg", "image.jpg")
  res = Net::HTTP.start(url.host, url.port) do |http|
    http.request(req)
  end
end
```

To post multiple files or attachments, simply include multiple parameters with
`UploadIO` values:

```ruby
require 'net/http/post/multipart'

url = URI.parse('http://www.example.com/upload')
req = Net::HTTP::Post::Multipart.new url.path,
  "file1" => UploadIO.new(File.new("./image.jpg"), "image/jpeg", "image.jpg"),
  "file2" => UploadIO.new(File.new("./image2.jpg"), "image/jpeg", "image2.jpg")
res = Net::HTTP.start(url.host, url.port) do |http|
  http.request(req)
end
```

To post files with other normal, non-file params such as input values, you need to pass hashes to the `Multipart.new` method.

In Rails 4 for example:

```ruby
def model_params
  require_params = params.require(:model).permit(:param_one, :param_two, :param_three, :avatar)
  require_params[:avatar] = model_params[:avatar].present? ? UploadIO.new(model_params[:avatar].tempfile, model_params[:avatar].content_type, model_params[:avatar].original_filename) : nil
  require_params
end

require 'net/http/post/multipart'

url = URI.parse('http://www.example.com/upload')
Net::HTTP.start(url.host, url.port) do |http|
  req = Net::HTTP::Post::Multipart.new(url, model_params)
  key = "authorization_key"
  req.add_field("Authorization", key) #add to Headers
  http.use_ssl = (url.scheme == "https")
  http.request(req)
end
```

Or in plain ruby:

```ruby
def params(file)
  params = { "description" => "A nice picture!" }
  params[:datei] = UploadIO.new(file, "image/jpeg", "image.jpg")
  params
end

url = URI.parse('http://www.example.com/upload')
File.open("./image.jpg") do |file|
  req = Net::HTTP::Post::Multipart.new(url.path, params(file))
  res = Net::HTTP.start(url.host, url.port) do |http|
    return http.request(req).body
  end
end
```
### Parts Headers
By default, all individual parts will include the header `Content-Disposition` as well as `Content-Length`, `Content-Transfer-Encoding` and `Content-Type` for the File Parts. 

You may optionally configure the headers `Content-Type` and `Content-ID` for both ParamPart and FilePart by passing in a `parts` header.

For example: 
```
url = URI.parse('http://www.example.com/upload')

params = {
  "file_metadata_01" => { "description" => "A nice picture!" },
  "file_content_01"  => UploadIO.new(file, "image/jpeg", "image.jpg")
}

headers = {
  'parts': {
    'file_metadata_01': {
      'Content-Type' => "application/json"
      }
    }
  }

req = Net::HTTP::Post::Multipart.new(uri, params, headers)
```
This would configure the `file_metadata_01` part to include `Content-Type` 

```
Content-Disposition: form-data; name="file_metadata_01"
Content-Type: application/json
  {
    "description" => "A nice picture!" 
  }
```

#### Custom Parts Headers
*For FileParts only.*

You can include any number of custom parts headers in addition to `Content-Type` and `Content-ID`. 
```
headers = {
  'parts': {
    'file_metadata_01': {
      'Content-Type' => "application/json",
      'My-Custom-Header' => "Yo Yo!"
      }
    }
  }
```



### Debugging

You can debug requests and responses (e.g. status codes) for all requests by adding the following code:

```ruby
http = Net::HTTP.new(uri.host, uri.port)
http.set_debug_output($stdout)
```

## Versioning

This library aims to adhere to [Semantic Versioning 2.0.0][semver].
Violations of this scheme should be reported as bugs. Specifically,
if a minor or patch version is released that breaks backward
compatibility, a new version should be immediately released that
restores compatibility. Breaking changes to the public API will
only be introduced with new major versions.

As a result of this policy, you can (and should) specify a
dependency on this gem using the [Pessimistic Version Constraint][pvc] with two digits of precision.

For example:

```ruby
spec.add_dependency 'multipart-post', '~> 2.1'
```

[semver]: http://semver.org/
[pvc]: http://guides.rubygems.org/patterns/#pessimistic-version-constraint

## License

Released under the MIT license.

Copyright, 2007-2013, by [Nick Sieger](https://nicksieger.com).  
Copyright, 2017, by [Samuel G. D. Williams](https://www.codeotaku.com).  
Copyright, 2019, by [Patrick Davey](https://psdavey.com).  

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
