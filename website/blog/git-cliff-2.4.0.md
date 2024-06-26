---
slug: 2.4.0
title: "What's new in 2.4.0? \U0001F195"
date: 2024-06-26T00:00:00.000Z
authors: orhun
tags:
  - release
---
<center>

  <a href="https://github.com/orhun/git-cliff">
    <img src="/img/git-cliff-anim.gif" />
  </a>

</center>

> [**git-cliff**](https://github.com/orhun/git-cliff) is a command-line tool (written in [Rust](https://www.rust-lang.org/)) that provides a highly customizable way to generate changelogs from git history.
>
> It supports using [custom regular expressions](/docs/configuration/git#commit_parsers) to alter changelogs which are mostly based on [conventional commits](/docs/configuration/git#conventional_commits). With a single [configuration file](/docs/configuration), a wide variety of formats can be applied for a changelog, thanks to the Jinja2/Django-inspired [template engine](/docs/category/templating).
>
> More information and examples can be found in the [GitHub repository](https://github.com/orhun/git-cliff).

---

### 🍵 Gitea Integration

`git-cliff` now supports integrating with repositories hosted on Gitea (e.g. [Codeberg](https://codeberg.org/) or your own instance!)

This means that you can now use the following variables in your changelog:

- Usernames (`${{ commit.gitea.username }}` or `${{ contributor.username }}`)
- Contributors list (`${{ gitea.contributors }}`)
- Pull requests (`${{ commit.gitea.pr_number }}` or `${{ contributor.pr_number }}`)

This means you can generate changelog entries like the following:

```md
## What's Changed

- feat(commit): add merge_commit flag to the context by @orhun in #389
- test(fixture): add test fixture for bumping version by @orhun in #360

## New Contributors

- @someone made their first contribution in #360
- @cliffjumper made their first contribution in #389

<!-- generated by git-cliff -->
```

To set up `git-cliff` for your project, simply:

1. Check out the [quickstart guide](https://git-cliff.org/docs/) for installation / initialization.
1. Set up the [Git remote](https://git-cliff.org/docs/configuration/remote/) for your GitLab project.
1. Update the changelog configuration to use the [template variables](https://git-cliff.org/docs/integration/gitea/).

:::tip

See the [Gitea integration](https://git-cliff.org/docs/integration/gitea) for detailed documentation and usage examples. It works very similar to the [GitHub integration](https://git-cliff.org/docs/integration/github).

:::

---

### 📤 Bump based on pattern

The `--bump` argument works as follows:

> - "fix:" -> increments `PATCH`
> - "feat:" -> increments `MINOR`
> - "scope!" (breaking changes) -> increments `MAJOR`

But what happens let's say you want to bump the major if the commit starts with "abc" instead?

Good news, `git-cliff` now supports bumping based on configurable custom patterns! Simply configure the following values in your configuration:

```toml
[bump]
custom_major_increment_regex = "abc"
custom_minor_increment_regex = "minor|more"
```

So with this commit history:

```sh
(HEAD -> main) abc: 1
(tag: 0.1.0) initial commit
```

The major will be bumped due to "abc" (`0.1.0` -> `1.0.0`)

```sh
$ git-cliff --bumped-version

1.0.0
```

---

### ⬆️ Set initial tag for bump

When using `--bump`, if there are no initial tags are found then the default used to be hardcoded as `0.1.0`.

Now you can configure this value in the configuration file as follows:

```toml
[bump]
initial_tag = "1.0.0"
```

You can also override this value from the command line as follows:

```sh
$ git-cliff --bump --tag=1.0.0
```

---

### ⚙️ `--ignore-tags` argument

The value of `[git.ignore_tags]` can now be overridden by the newly added `--ignore-tags` argument:

```sh
$ git-cliff --ignore-tags "rc|v2.1.0|v2.1.1"
```

is the equivalent of:

```toml
[git]
# regex for ignoring tags
ignore_tags = "rc|v2.1.0|v2.1.1"
```

---

### 📝 Header template

`[changelog.header]` is now a template similar to `body` and `footer`. It used to be a raw string value that is added to the top of the changelog but now you can use the template variables and functions in it!

For example:

```toml
[changelog]
# template for the changelog footer
header = """
# Changelog
{% for release in releases %}\
    {% if release.version %}\
        {% if release.previous.version %}\
            <!--{{ release.previous.version }}..{{ release.version }}-->
        {% endif %}\
    {% else %}\
        <!--{{ release.previous.version }}..HEAD-->
    {% endif %}\
{% endfor %}\
"""
```

Will result in:

```md
# Changelog

<!--v3.0.0..HEAD-->
<!--v0.2.0..v3.0.0-->
<!--v0.1.0..v0.2.0-->
```

<sup> There is a [pending issue](https://github.com/orhun/git-cliff/issues/712) that needs fixing for `--prepend` to work with header template. </sup>

---

### 🧮 Parse commits by footer

You can now parse the commits by their footer value!

Let's say you want to skip this commit:

```sh
git commit -m "test: add more tests" -m "changelog: ignore"
```

This is now possible:

```toml
[git]
# regex for parsing and grouping commits
commit_parsers = [
    { footer = "^changelog: ?ignore", skip = true },
]
```

---

### 📩 Support tag messages

You can now include the tag messages (of release tags) in your changelog!

This can be useful for having "headlines" for a release like so:

```md
## [1.0.1] - 2021-07-18

This is the release-tag message
```

The message is available in the context of the template as the `{{ message }}` variable:

```
{% if message %}
    {{ message }}
{% endif %}\
```

You can also override the tag message for the unreleased changes via `--with-tag-message` argument as follows:

```sh
$ git cliff --bump --unreleased --with-tag-message "This is the release-tag message"
```

The recommended way of setting tag messages is to use annotated tags in your project:

```sh
$ git tag v1.0.0 -m "This is the release-tag message"
```

---

### 📂 Repository path in context

You can now use `{{ repository }}` variable in the template to get the repository path:

```
## Release [{{ version }}] - {{ timestamp | date(format="%Y-%m-%d") }} - {{ repository }}
```

This is especially useful when you use `git-cliff` with multiple repositories. (e.g. `git-cliff -r repo1 -r repo2`)

---

### 📡 Remote data in context

We updated the changelog processing order to make remote data (e.g. GitHub commits, pull requests, etc.) available in the context.

For example:

```sh
$ git cliff --github-repo orhun/git-cliff -c examples/github.toml --no-exec -u -x
```

This command will now contain the GitHub data such as:

```json
 "github": {
      "contributors": [
        {
          "username": "bukowa",
          "pr_title": "style(lint): fix formatting",
          "pr_number": 702,
          "pr_labels": [],
          "is_first_time": true
        },
    ],
}
```

---

### 🧰 Other

- _(website)_ Add information about `--bump` with `tag prefixes` ([#695](https://github.com/orhun/git-cliff/issues/695)) - ([4cd18c2](https://github.com/orhun/git-cliff/commit/4cd18c2bcdb2ce21234776364598d42261df004d))
- _(fixture)_ Support running fixtures on mingw64 ([#708](https://github.com/orhun/git-cliff/issues/708)) - ([dabe716](https://github.com/orhun/git-cliff/commit/dabe716c201fedf3021d89c5a8564794bda07f2a))

---

## Contributions 👥

- @MeitarR made their first contribution in [#713](https://github.com/orhun/git-cliff/pull/713)
- @bukowa made their first contribution in [#696](https://github.com/orhun/git-cliff/pull/696)
- @Cyclonit made their first contribution in [#698](https://github.com/orhun/git-cliff/pull/698)
- @jan-ferdinand made their first contribution in [#569](https://github.com/orhun/git-cliff/pull/569)
- @Theta-Dev made their first contribution in [#680](https://github.com/orhun/git-cliff/pull/680)
- @tcarmet made their first contribution in [#694](https://github.com/orhun/git-cliff/pull/694)

Any contribution is highly appreciated! See the [contribution guidelines](https://github.com/orhun/git-cliff/blob/main/CONTRIBUTING.md) for getting started.

Feel free to [submit issues](https://github.com/orhun/git-cliff/issues/new/choose) and join our [Discord](https://discord.gg/W3mAwMDWH4) / [Matrix](https://matrix.to/#/#git-cliff:matrix.org) for discussion!

Follow `git-cliff` on [Twitter](https://twitter.com/git_cliff) & [Mastodon](https://fosstodon.org/@git_cliff) to not miss any news!

## Support 🌟

If you liked `git-cliff` and/or my other projects [on GitHub](https://github.com/orhun), consider [donating](https://donate.orhun.dev) to support my open source endeavors.

- 💖 GitHub Sponsors: [@orhun](https://github.com/sponsors/orhun)
- ☕ Buy Me A Coffee: [https://www.buymeacoffee.com/orhun](https://www.buymeacoffee.com/orhun)

Have a fantastic day! ⛰️