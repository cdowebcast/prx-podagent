# PRX Podcast User-Agent Parser

[![license](https://img.shields.io/github/license/PRX/prx-podagent.svg)](LICENSE)
[![npm](https://img.shields.io/npm/v/prx-podagent.svg)](https://www.npmjs.com/package/prx-podagent)
[![npm](https://img.shields.io/npm/dt/prx-podagent.svg)](https://www.npmjs.com/package/prx-podagent)
[![build status](https://travis-ci.org/PRX/prx-podagent.svg?branch=master)](https://travis-ci.org/PRX/prx-podagent)

**[prx.github.io/prx-podagent](https://prx.github.io/prx-podagent/)**

## Description

Basic User-Agent string parser that includes some basic podcasting apps.  This
project is intended to help parse/group requests for analytics purposes, not for
browser feature detection.

The included `agents.lock.yml` also includes name/type/os IDs, in case you want
to normalize the strings in your database.

## Install

Just `npm install --save prx-podagent`. Or to use outside of node, just grab a
database file out of the `db/` directory.

## Usage

```node
const podagent = require('prx-podagent');
const agent = podagent.parse('some-string');
if (agent) {
  console.log('Match:', agent.name, agent.type, agent.os);
} else {
  console.log('Did not match any known agents');
}
```

Or to DIY use the json lock file:

```javascript
  $.getJSON('agents.json', function(db) {
    db.agents.forEach(function(a) {
      a.regex = new RegExp(a.regex, a.ignorecase ? 'i' : undefined);
    });

    var str = 'some-agent-string';
    var agent = db.agents.find(function(a) { return a.regex.test(str); });
    if (agent) {
      console.log('matched!', agent);
    } else {
      console.log('no match for:', str);
    }
  });
```

Or in Ruby:

```ruby
require 'yaml'

DB = YAML.load_file('db/agents.lock.yml')
def match_agent(str)
  DB['agents'].find { |a| Regexp.new(a['regex'], a['ignorecase'] ? 'i' : nil).match(str) }&.tap do |match|
    %w(name type os).each { |k| match[k] = DB['tags'][match[k]] }
  end
end

puts match_agent('Pandora/1812.2 Android/5.1.1 ford (ExoPlayerLib2.8.2)').inspect
# {"regex"=>"/^HardCast.+CFNetwork/", "name"=>"HardCast", "type"=>"Mobile App", "os"=>"iOS"}
puts match_agent('blah blah blah').inspect
# nil
```

Or in PHP:

```php
<?php
require_once 'spyc.php';
$DB = Spyc::YAMLLoad('db/agents.lock.yml');

function match_agent($str) {
  global $DB;

  $match = NULL;
  foreach ($DB['agents'] as $agent) {
    $pattern = '/' . $agent['regex'] . '/' . ($agent['ignorecase'] ? 'i' : '');
    if (preg_match($pattern, $str)) {
      $match = $agent;
      foreach (array('name', 'type', 'os') as $key) {
        $match[$key] = $DB['tags'][$match[$key]];
      }
      break;
    }
  }
  return $match;
}

var_dump(match_agent('Pandora/1812.2 Android/5.1.1 ford (ExoPlayerLib2.8.2)'));
?>
```

## Development

Basic tests are located in the `/test` directory, and can be run with `npm test`.

Additionally, there is a `test/support/testagents.csv` file containing some actual
production logs.  The "coverage" and "omissions" tests use this file to check that
the database file accounts for all the major known user agents.

To add a new user agent:

1. Edit the `db/agents.yml` file to include your new regular expression, plus
   an example user-agent string or two.
2. Run `npm test` (or just `mocha test/examples-test.js`) to test that your
   example strings match the regexp.
3. Run `npm lock` to regenerate the `db/agents.lock.yml`.  This file normalizes
   the text tags/labels shared between the various matchers.  Check that your
   change didn't add any unexpected new tags (if you accidentally changed the
   case of a label or something).
4. Bump the patch version (rightmost) in `package.json` and create a pull
   request to this repo.

## License

[MIT License](LICENSE)

## Contributing

1. Fork it
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create new Pull Request
