# Scheduled post action

A GitHub action that creates a scheduled post from data files generated by [read-action](https://github.com/library-pals/read-action), [bookmark-action](https://github.com/library-pals/bookmark-action), and [spotify-to-yaml-action](https://github.com/library-pals/spotify-to-yaml-action).

If you're including playlist data generated by the spotify-to-yaml-action, make sure it's schedule to run before this action.

<!-- START GENERATED DOCUMENTATION -->

## Set up the workflow

To use this action, create a new workflow in `.github/workflows` and modify it as needed:

```yml
name: Scheduled post

# Grant the action permission to write to the repository
permissions:
  contents: write
  pull-requests: write

on:
  schedule:
    - cron: "00 02 20 Mar,Jun,Sep,Dec *"

jobs:
  scheduled-post:
    runs-on: ubuntu-latest
    name: Write scheduled post
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Write scheduled post
        uses: library-pals/scheduled-post-action@v0.0.0
        with:
          github-username: katydecorah
          github-repository: archive
          source-bookmarks: recipes|_data/recipes.json
        env:
          TOKEN: ${{ secrets.TOKEN }}
      - name: Commit files
        run: |
          git pull
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A && git commit -m "${{ env.title }}"
          git push
```

### Additional example workflows

<details>
<summary>Seasonal scheduled post</summary>

```yml
name: Seasonal scheduled post

on:
  workflow_dispatch:
  schedule:
    - cron: "00 02 20 Mar,Jun,Sep,Dec *"

jobs:
  scheduled-post:
    runs-on: ubuntu-latest
    name: Write scheduled post
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set post title and dates
        run: |
          MONTH=$(date +%m)
          YEAR=$(date +%Y)

          declare -A SEASONS=(
            ["03"]="Winter"
            ["06"]="Spring"
            ["09"]="Summer"
            ["12"]="Fall"
          )

          set_env_vars() {
            local season=$1
            local start_date=$2
            local end_date=$3
            local post_title=""

            if [ "$season" = "Winter" ]; then
              post_title="$(($YEAR - 1))/${YEAR} ${season}"
            else
              post_title="${YEAR} ${season}"
            fi

            echo "POST_TITLE=${post_title}" >> $GITHUB_ENV
            echo "START_DATE=${start_date}" >> $GITHUB_ENV
            echo "END_DATE=${end_date}" >> $GITHUB_ENV
          }

          case $MONTH in
            "03")
              set_env_vars ${SEASONS[$MONTH]} "$(($YEAR - 1))-12-21" "${YEAR}-03-20"
              ;;
            "06")
              set_env_vars ${SEASONS[$MONTH]} "${YEAR}-03-21" "${YEAR}-06-20"
              ;;
            "09")
              set_env_vars ${SEASONS[$MONTH]} "${YEAR}-06-21" "${YEAR}-09-20"
              ;;
            "12")
              set_env_vars ${SEASONS[$MONTH]} "${YEAR}-09-21" "${YEAR}-12-20"
              ;;
          esac
      - name: Write scheduled post
        uses: library-pals/scheduled-post-action@v0.0.0
        with:
          github-username: katydecorah
          github-repository: archive
          source-bookmarks: recipes|_data/recipes.json
          book-tags: "recommend,skip"
          start-date: ${{ env.START_DATE }}
          end-date: ${{ env.END_DATE }}
          post-title: ${{ env.POST_TITLE }}
        env:
          TOKEN: ${{ secrets.TOKEN }}
      - name: Commit files
        run: |
          git pull
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A && git commit -m "${{ env.POST_TITLE }}"
          git push
```

</details>

<details>
<summary>Uses a custom markdown template (`post-template`) and customizes the `posts-directory`, with a manual workflow trigger.</summary>

```yml
name: Uses a custom markdown template (`post-template`) and customizes the `posts-directory`, with a manual workflow trigger.

on:
  workflow_dispatch:
    inputs:
      start-date:
        description: "The start date for the post in the format YYYY-MM-DD"
        type: string
        required: true
      end-date:
        description: "The end date for the post in the format YYYY-MM-DD"
        type: string
        required: true
      post-title:
        description: "The title of the post"
        type: string
        required: true

jobs:
  scheduled-post:
    runs-on: ubuntu-latest
    name: Write scheduled post
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Write scheduled post
        uses: library-pals/scheduled-post-action@v0.0.0
        with:
          github-username: katydecorah
          github-repository: archive
          post-template: .github/actions/post-template-basic.md
          posts-directory: books/
          source-bookmarks: recipes|_data/recipes.json
        env:
          TOKEN: ${{ secrets.TOKEN }}
      - name: Commit files
        run: |
          git pull
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A && git commit -m "${{ env.title }}"
          git push
```

</details>

## Action options

- `github-username`: Required. The GitHub username that owns the repository with the data files.

- `github-repository`: Required. The Github repository that has the data files.

- `posts-directory`: The path to where you want to save your scheduled post files to in this repository. Default: `notes/_posts/`.

- `post-template`: If you'd like to customize the [markdown template](src/template.md), define a path to your own. Example: `post-template: .github/actions/post-template.md`. The markdown template shows all the available variables and an idea for how you may want to format this file. For now, the templating is simplistic and does not offer functionality outside of this action replacing variable names.

- `source-books`: Define the label and file path for the books data generated by [read-action](https://github.com/katydecorah/read-action). Separate the label and file path with a pipe (`|`). If you do not have books, set this value to `false`. Note: this value will **not** change the variable name in the markdown template, which is `bookYaml` and `bookMarkdown`. Default: `books|_data/read.json`.

- `source-bookmarks`: Define the label and file path for the bookmarks data generated by [bookmark-action](https://github.com/katydecorah/bookmark-action). Separate the label and file path with a pipe (`|`). If you do not have bookmarks, set this value to `false`. Note: this value will **not** change the variable name in the markdown template, which is `bookmarkYaml` and `bookmarkMarkdown`. Default: `bookmarks|_data/bookmarks.json`.

- `source-playlist`: Define the file path for the playlist data generated by [spotify-to-yaml-action](https://github.com/katydecorah/spotify-to-yaml-action). If you do not have playlists, set this value to `false`. Default: `_data/playlists.yml`.

- `book-tags`: Allow specific tags to be passed through. Separate each tag with a comma.

- `start-date`: The start date for the post. The format is `YYYY-MM-DD`. This can be set as an action input or workflow input.

- `end-date`: The end date for the post. The format is `YYYY-MM-DD`. This can be set as an action input or workflow input.

- `post-title`: The title of the post. This can be set as an action input or workflow input.
<!-- END GENERATED DOCUMENTATION -->
