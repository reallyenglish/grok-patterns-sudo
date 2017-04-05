# grok-patterns-sudo

`grok` patterns for `sudo(8)` based on
[whyscream/postfix-grok-patterns](https://github.com/whyscream/postfix-grok-patterns).

## Usage

```sh
git clone git@github.com:reallyenglish/grok-patterns-sudo.git
cd grok-patterns-sudo
bundle install --path vendor/bundle
git submodule update --init --recursive
```

## Creating a pattern

### Creating a test for the pattern

```sh
vim test/sudo_${TEST_NUMBER}.yaml
```

The test file is a YAML file. `pattern` is the pattern name to test, `data` is
the input data for the pattern. `results` is a hash and should have variables
as key, and values that are expected.

```yaml
pattern: ^%{SUDO}$
data: "nagios : TTY=unknown ; PWD=/ ; USER=root ; COMMAND=/usr/sbin/mptutil show volumes"
results:
  sudo_user: nagios
  sudo_tty: unknown
  sudo_pwd: /
  sudo_runas: root
  sudo_command: /usr/sbin/mptutil show volumes
```

### Adding the pattern

```sh
vim patterns/sudo.grok
(... add your pattern ...)
```
An example:

```
SUDO_TTY TTY=%{NOTSPACE:sudo_tty}
SUDO_PWD PWD=%{DATA:sudo_pwd}
SUDO_COMMAND COMMAND=%{DATA:sudo_command}
SUDO_USER %{NOTSPACE:sudo_user}
SUDO_RUNAS USER=%{SUDO_USER:sudo_runas}

SUDO_COMMAND_SUCCESSFUL %{SUDO_USER} : %{SUDO_TTY} ; %{SUDO_PWD} ; %{SUDO_RUNAS} ; %{SUDO_COMMAND}
SUDO_INFO %{SUDO_COMMAND_SUCCESSFUL}
SUDO %{SUDO_INFO}|%{SUDO_ERROR}
```
For details about `grok` filter, see [the official
documentation](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).


### Testing

```sh
bundle exec rake test
```

## Variables

A variable must be prefixed with `sudo_`.

### Reserved variables

These are commonly used by the patterns, and reserved.

| Name | Description | Required? |
|------|-------------|-----------|
| `sudo_command` | command passed to `sudo(8)` | Optional |
| `sudo_message` | message that describes logged events | Optional |
| `sudo_pwd`     | directory where `sudo(8)` was executed | Optional |
| `sudo_reason`  | reason of the action taken | Optional |
| `sudo_runas`   | target user of `sudo(8)` | Optional |
| `sudo_tty`     | `tty(4)` name | Optional |
| `sudo_user`    | user that invoked `sudo(8)` | Optional |

A variable MAY be assigned by multiple patterns. An example is
`sudo_command`.

## Pattern name

### Reserved pattern names

| Name | Description |
|------|-------------|
| `SUDO_INFO` | a regexp of harmless, noisy, or uninteresting log messages, `OR`ed by `|` |
| `SUDO_ERROR` | a regexp of log messages that you should watch |
| `SUDO` | a regexp of `SUDO_INFO` and `SUDO_ERROR` `OR`ed by `|` |

Note that `SUDO_INFO` and `SUDO_ERROR` have nothing to do with log levels in
`syslog(2)`. They are simply for pattern authors to categorise patterns.

A pattern name must be prefixed with `SUDO_INFO_` or `SUDO_ERROR_`.

Use common postfix in pattern name. If `sudo_message` is defined by
`SUDO_MESSAGE_TIMEOUT`, the pattern should be either `SUDO_INFO_TIMEOUT` or
`SUDO_ERROR_TIMEOUT`, depending on the importance of the event.

Every pattern must be listed in either `SUDO_INFO` or `SUDO_ERROR`.

## LICENSE

```
Copyright 2016 Tom Hendrikx <tom@whyscream.net>
          2017 Tomoyuki Sakurai <tomoyukis@reallyenglish.com>

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
this list of conditions and the following disclaimer in the documentation
and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors
may be used to endorse or promote products derived from this software without
specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
