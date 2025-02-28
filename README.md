# Forked because of mimemagic versions being yanked
Until an official fix is released, i've created this fork where mimemagic is updated to version 0.4.1, which is available under an MIT license rather than mimemagic 0.3.3 which 1) does not exist anymore and 2) was distributed under what should have been a GPL-2 license. Feel free to use this version (at your own risk). I will delete the fork once an official fix is released. I do not claim the be the author of this package, nor do i claim any ownership of the content here. If for whatever reason you disagree with anything in this version, please let me know and i will take it down again.

# Marcel

Marcel attempts to choose the most appropriate content type for a given file by looking at the binary data, the filename, and any declared type (perhaps passed as a request header):

It's used like this:

```ruby
Marcel::MimeType.for Pathname.new("example.gif")
#  => "image/gif"

File.open "example.gif" do |file|
  Marcel::MimeType.for file
end
#  => "image/gif"

Marcel::MimeType.for Pathname.new("unrecognisable-data"), name: "example.pdf"
#  => "application/pdf"

Marcel::MimeType.for extension: ".pdf"
#  => "application/pdf"

Marcel::MimeType.for Pathname.new("unrecognisable-data"), name: "example", declared_type: "image/png"
#  => "image/png"

Marcel::MimeType.for StringIO.new(File.read "unrecognisable-data")
#  => "application/octet-stream"
```

By preference, the magic number data in any passed in file is used to determine the type. If this doesn't work, it uses the type gleaned from the filename, extension, and finally the declared type. If no valid type is found in any of these, "application/octet-stream" is returned.

Some types aren't easily recognised solely by magic number data. For example Adobe Illustrator files have the same magic number as PDFs (and can usually even be viewed in PDF viewers!). For these types, Marcel uses both the magic number data and the file name to work out the type:

```ruby
Marcel::MimeType.for Pathname.new("example.ai"), name: "example.ai"
#  => "application/illustrator"
```

This only happens when the type from the filename is a more specific type of that from the magic number. If it isn't the magic number alone is used.

```ruby
Marcel::MimeType.for Pathname.new("example.png"), name: "example.ai"
#  => "image/png"
# As "application/illustrator" is not a more specific type of "image/png", the filename is ignored
```

## Motivation

Marcel was extracted from Basecamp 3, in order to make our file detection logic both easily reusable but more importantly, easily testable. Test fixtures have been added for all of the most common file types uploaded to Basecamp, and other common file types too. We hope to expand this test coverage with other file types as and when problems are identified.

## Implementation

At present, marcel is mainly a thin wrapper around the [mimemagic gem][mime-magic-gem-url]. It adds priority logic (preferring magic over name when given both), some extra type definitions, and common type subclasses (including Keynote, Pages, etc).

## Testing

The main test fixture files are split into two folders, those that can be recognised by magic numbers, and those that can only be recognised by name. Even though strictly unnecessary, the fixtures in both folders should all be valid files of the type they represent.

## References

* [Mimemagic gem][mime-magic-gem-url]
* [Shared MIME-info Database Specification](https://specifications.freedesktop.org/shared-mime-info-spec/latest/)

[mime-magic-gem-url]: https://github.com/minad/mimemagic
