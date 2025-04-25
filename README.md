# poc-itamae-ruby-3-4-bug

[github.com/itamae-kitchen#297](https://github.com/itamae-kitchen/itamae/pull/297) causes CI to fail only in Ruby 3.4.

This repository is for verifying the above bug.

## About

All tests pass in Ruby 3.3, but the following error occurs in Ruby 3.4:

<details>

<summary>Ruby 3.4 Serverspec log</summary>

### Ruby 3.4 Serverspec log

```
File "/tmp/file_edit_with_content_change_updates_timestamp"
  mtime
    is expected to be > 2016-05-02T01:23:45+00:00 (FAILED - 1)
```

```
File "/tmp/file_with_content_change_updates_timestamp"
  mtime
    is expected to be > 2016-05-01T01:23:45+00:00 (FAILED - 2)
```

```
Failures:

  1) File "/tmp/file_edit_with_content_change_updates_timestamp" mtime is expected to be > 2016-05-02T01:23:45+00:00
     Failure/Error: its(:mtime) { should be > DateTime.iso8601("2016-05-02T01:23:45Z") }
       expected: > #<DateTime: 2016-05-02T01:23:45+00:00 ((2457511j,5025s,0n),+0s,2299161j)>
            got:   #<DateTime: 1980-01-02T09:00:00+09:00 ((2444241j,0s,0n),+32400s,2299161j)>

     # ./spec/integration/default_spec.rb:287:in 'block (2 levels) in <top (required)>'

  2) File "/tmp/file_with_content_change_updates_timestamp" mtime is expected to be > 2016-05-01T01:23:45+00:00
     Failure/Error: its(:mtime) { should be > DateTime.iso8601("2016-05-01T01:23:45Z") }
       expected: > #<DateTime: 2016-05-01T01:23:45+00:00 ((2457510j,5025s,0n),+0s,2299161j)>
            got:   #<DateTime: 1980-01-02T09:00:00+09:00 ((2444241j,0s,0n),+32400s,2299161j)>

     # ./spec/integration/default_spec.rb:319:in 'block (2 levels) in <top (required)>'

Finished in 3.06 seconds (files took 0.23664 seconds to load)
153 examples, 2 failures, 2 pending

Failed examples:

rspec ./spec/integration/default_spec.rb:287 # File "/tmp/file_edit_with_content_change_updates_timestamp" mtime is expected to be > 2016-05-02T01:23:45+00:00
rspec ./spec/integration/default_spec.rb:319 # File "/tmp/file_with_content_change_updates_timestamp" mtime is expected to be > 2016-05-01T01:23:45+00:00
```

</details>

This is the code where I think the problem is occurring.

<details>

<summary>The code where the error occurs</summary>

### The code where the error occurs

[itamae/spec/integration/recipes/default.rb#L458-L465](https://github.com/itamae-kitchen/itamae/blob/57dfb50d93c836e77911bf3a592be8bd56ec21ba/spec/integration/recipes/default.rb#L458-L465)

```ruby:itamae/spec/integration/recipes/default.rb
execute "f=/tmp/file_edit_with_content_change_updates_timestamp && echo 'Hello, world' > $f && touch -d 2016-05-02T01:23:45Z $f"

file "/tmp/file_edit_with_content_change_updates_timestamp" do
  action :edit
  block do |content|
    content.gsub!('world', 'Itamae')
  end
end
```

[itamae/spec/integration/recipes/default.rb#L498-L502](https://github.com/itamae-kitchen/itamae/blob/57dfb50d93c836e77911bf3a592be8bd56ec21ba/spec/integration/recipes/default.rb#L498-L502)

```ruby:itamae/spec/integration/recipes/default.rb
execute "f=/tmp/file_edit_with_content_change_updates_timestamp && echo 'Hello, world' > $f && touch -d 2016-05-02T01:23:45Z $f"

file "/tmp/file_edit_with_content_change_updates_timestamp" do
  action :edit
  block do |content|
    content.gsub!('world', 'Itamae')
  end
end
```

[itamae/spec/integration/default_spec.ruby#L275-L277](https://github.com/itamae-kitchen/itamae/blob/57dfb50d93c836e77911bf3a592be8bd56ec21ba/spec/integration/default_spec.rb#L275-L277)

```ruby:itamae/spec/integration/default_spec.ruby
describe file('/tmp/file_edit_with_content_change_updates_timestamp') do
  its(:mtime) { should be > DateTime.iso8601("2016-05-02T01:23:45Z") }
end
```

[itamae/spec/integration/default_spec.ruby#L307-L309](https://github.com/itamae-kitchen/itamae/blob/57dfb50d93c836e77911bf3a592be8bd56ec21ba/spec/integration/default_spec.rb#L307-L309)

```ruby:itamae/spec/integration/default_spec.ruby
describe file('/tmp/file_with_content_change_updates_timestamp') do
  its(:mtime) { should be > DateTime.iso8601("2016-05-01T01:23:45Z") }
end
```

</details>

## TODO

- [x] Porting the itamae repository
- [ ] Extract only the part where the error occurs
- [ ] Minimal repro code
