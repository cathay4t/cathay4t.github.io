---
---

# Local note taking system using vim + markdown + jekyll

Real appreciate the great note taking software [basket][1] created
by Sébastien Laoût and others. It's the only qt3 software I am still
using for 7 years. After created github pages for [libstoragemgmt
documentation][2], I decided to move on by creating my own note taking
system using vim + markdown + Jekyll.

I might take some confidential notes which I would rather not to share
via github public repo, but github pages is publicly accessible for
everyone. So this document will show you how to run an local private
git pages and optimize it for note taking.


## How it looks

* Index page
  ![index-page](../../images/2015/01_index_page.png)
* Note page
  ![note-page](../../images/2015/01_note_page.png)

## How it works

Basically, it's just a local running Jekyll, you could refer to
[github help page: Using Jekyll with Pages][3] for detail.

Quick notes:

* Use vim to create markdown file using [github favored markdown
  format][4].
* Jekyll follow the configuration in `_config.yml` to convert your
  markdown files into html files.


## How to build it
I am using RHEL 7 for the flowing examples.

### Prerequisite
```bash
    # Install ruby and bundle
    sudo yum install rubygem-bundler -y

    # Install screen which we used to start http daemon
    sudo yum install screen -y

    # Create folder "~/notes" to hold all the files
    mkdir ~/notes && cd ~/notes
    echo "source 'https://rubygems.org'
    gem 'github-pages'" > Gemfile
    bundle install
```

### Choose page layout.
You could manually create your own template, but for now, we use
github pages generator.

1. Create new git repository
  [Create new git repository][5] just for temp use, you could delete
  if you don't want to share your notes to public.
  I am using git repo `cathay4t/tmp_for_gp` as an example.

1. Goes to setting page of the new repo.
  Example link is https://github.com/cathay4t/tmp_for_gp/settings

1. Click "Automatic page generator" button in the setting page.

1. Choose the project name and click "continue to layouts" at the
   bottom, no need to edit the body messages yet.

1. Choose your favorite layout.

### Customize local Jekyll
1. Pull github files to local.

    ```bash
    git clone https://github.com/cathay4t/tmp_for_gp.git -o notes
    # Now you can delete github repo if you don't want it.
    ```
1. Run local website:
  Create `~/bin/jk_note` like this:

    ```bash
    #!/bin/bash
    screen -d -m -S jk_note \
        /bin/bash -c \
        "cd ~/notes/ && \
         bundle exec jekyll serve --host 127.0.0.1 -P 8080 --watch"
    ```
  You note webpage will be at `http://localhost:8080`.
  The `--detach` option does not works with `--watch`, so we need
  screen here. Check [this link][6] for more detail.
  You might want to add this script to on boot also.

1. Configure Jekyll to use github favored markdown.
  Edit `~/notes/_config.yml` like this

    ```
    markdown: redcarpet
    redcarpet:
        extensions: ["tables", "autolink", "strikethrough",
                     "space_after_headers", "with_toc_data",
                     "fenced_code_blocks"]

    defaults:
      -
        scope:
          path: ""
          # an empty string here means all files in the project
        values:
          layout: "default"
          # This means markdown files will use _layouts/default.html
          # for converting markdown to html by replacing
          # {{ content }} with markdown to html strings.
    ```

1. Customize layout html `_layouts/default.html`.
    * Change css link to absolute paths like this:

        ```html
        {% raw %}
        <link
            rel="stylesheet" type="text/css"
            href="{{ site.github.url }}/stylesheets/stylesheet.css"
            media="screen">
        <link
            rel="stylesheet" type="text/css"
            href="{{ site.github.url }}/stylesheets/pygment_trac.css"
            media="screen">
        <link
            rel="stylesheet" type="text/css"
            href="{{ site.github.url }}/stylesheets/print.css"
            media="print">
        {% endraw %}
        ```
    * Provide an command at the head for quick editing:

        ```html
        {% raw %}
        <pre><code
                class="language-bash" data-lang="bash"
                >konsole -e vim ~/notes/{{ page.path }}</code
                ></pre>
        {% endraw %}
        ```
        With this, everytime you want to edit this page, just
        copy the command on the head, then "alt+F2", paste, enter.

    * Create `index.md` at the top folder.
      Create `index.md` like this to include all sub notes:

        ```
        ---
        title: Notes of Gris Ge
        ---

        ## 1. Linux


        ## 2. Dev
         * [LSM][21]

        ## 9. Misc
         * [tmp][91]
         * [note][92]

        [1]: linux/
        [21]: dev/lsm.html
        [91]: misc/tmp.html
        [92]: misc/note.html
        ```
        The heading two `---` is required. Check [this link][8].
        The varible `title` defined between `---` could be used
        in layout html as `{% raw %}{{ page.title }}{% endraw %}`.

1. Porting your existing notes to markdowns.


## Tips
* [Display Liquid code in Jekyll][9]

```
{% raw %}
# Escape {{
{% endraw %}

{{ "{% raw " }} %}
{% raw %}
Hello, my name is {{name}}.
{% endraw %}

{{ "{% endraw " }} %}

# Escape {{ "{%" }}
{% raw %}
{{ "{% this " }}%}
{% endraw %}
```

* Quickly note something down.
  Create `~/bin/tn` and assign a shortcut to this script:

```bash
#!/bin/bash

konsole -e vim ~/notes/misc/tmp.md
```

* Show markdown headers in vim taglist
  If you are [taglist][10] user, create `~/.ctags` like this:

```
--recurse=yes
--tag-relative=yes
--exclude=.git

--langdef=markdown
--langmap=markdown:.md
--regex-markdown=/^#[ ]+(.*)/\1/h,heading1/
--regex-markdown=/^##[ ]+(.*)/\1/h,heading2/
--regex-markdown=/^###[ ]+(.*)/.\1/h,heading3/
--regex-markdown=/^####[ ]+(.*)/. \1/h,heading4/
--regex-markdown=/^#####[ ]+(.*)/.  \1/h,heading5/
--regex-markdown=/^######[ ]+(.*)/.   \1/h,heading6/
--regex-markdown=/^#######[ ]+(.*)/.    \1/h,heading7/
```

* Use git to trace all changes.

  As this is note taking system, I don't want to do manually git
  commit on my every edit, let's use crontab to do it for ourslef.
  Create script `~/bin/jk_note_git_auto_commit.sh`:

```bash
#!/bin/bash
cd ~/notes
git add ./
X="$(git status|grep 'nothing to commit')"
if [ "CHK${X}" == "CHK" ];then
    git commit -a -m"`date '+%Y-%m-%d %H:%M:%S UTC%:::z'`"
fi
```
  Add this line to `crontab -e`:

```
*/30 * * * * /home/fge/bin/jk_note_git_auto_commit.sh
```

* Sync to other location.

  There are many ways to sync your local notes to remote sites.
  You could use `git push` by backing up your local notes to remote
  repositories or use `rsync` for notes folder backup or other 1000+
  ways.

  **PLEASE DO BACKUP YOUR DATA!!!**

  **PLEASE DO BACKUP YOUR DATA!!!**

  **PLEASE DO BACKUP YOUR DATA!!!**

  **PLEASE DO BACKUP YOUR DATA!!!**

  **PLEASE DO BACKUP YOUR DATA!!!**

  **PLEASE CREATE OFFLINE BACKUP ALSO!!!**

  **PLEASE CREATE OFFLINE BACKUP ALSO!!!**

  **PLEASE CREATE OFFLINE BACKUP ALSO!!!**

  **PLEASE CREATE OFFLINE BACKUP ALSO!!!**


## Useful Links

* [Jekyllrb document][11]
* [GitHub: Using Jekyll with Pages][12]

[1]: http://basket.kde.org/
[2]: http://libstorage.github.io/libstoragemgmt-doc/
[3]: https://help.github.com/articles/using-jekyll-with-pages/
[4]: https://help.github.com/articles/github-flavored-markdown/
[5]: https://github.com/new
[6]: http://stackoverflow.com/questions/22898039/jekyll-serve-watch-doesnt-work-in-combination-with-detach
[8]: http://jekyllrb.com/docs/frontmatter/
[9]: https://truongtx.me/2013/01/09/display-liquid-code-in-jekyll/
[10]: http://vim-taglist.sourceforge.net/
[11]: http://jekyllrb.com/docs/home/
[12]: https://help.github.com/articles/using-jekyll-with-pages/
