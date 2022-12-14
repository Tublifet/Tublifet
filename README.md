

<h1 align="center">Hi 👋, I'm Nguyen Mai Thanh Truc</h1>
<p align="left">
</p>

<h3 align="left">Languages and tools:</h3>
<p align="left"> <a href="https://www.w3schools.com/cpp/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/cplusplus/cplusplus-original.svg" alt="cplusplus" width="40" height="40"/> </a> <a href="https://www.linux.org/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/linux/linux-original.svg" alt="linux" width="40" height="40"/> </a> <a href="https://www.microsoft.com/en-us/sql-server" target="_blank" rel="noreferrer"> <img src="https://www.svgrepo.com/show/303229/microsoft-sql-server-logo.svg" alt="mssql" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> </p>

name: main

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          cache: yarn
          node-version: 16
      - run: yarn install --frozen-lockfile

      - run: yarn type
      - run: yarn lint
      - run: yarn test --ci

  test-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: update action.yml to use image from local Dockerfile
        run: |
          sed -i "s/image: .*/image: Dockerfile/" action.yml

      - name: generate-snake-game-from-github-contribution-grid
        id: generate-snake
        uses: ./
        with:
          github_user_name: platane
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark
            dist/github-contribution-grid-snake.gif?color_snake=orange&color_dots=#bfd6f6,#8dbdff,#64a1f4,#4b91f1,#3c7dd9

      - name: ensure the generated file exists
        run: |
          ls dist
          test -f dist/github-contribution-grid-snake.svg
          test -f dist/github-contribution-grid-snake-dark.svg
          test -f dist/github-contribution-grid-snake.gif

      - uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  test-action-svg-only:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          cache: yarn
          node-version: 16
      - run: yarn install --frozen-lockfile

      - name: build svg-only action
        run: |
          yarn build:action
          rm -r svg-only/dist
          mv packages/action/dist svg-only/dist

      - name: generate-snake-game-from-github-contribution-grid
        id: generate-snake
        uses: ./svg-only
        with:
          github_user_name: platane
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: ensure the generated file exists
        run: |
          ls dist
          test -f dist/github-contribution-grid-snake.svg
          test -f dist/github-contribution-grid-snake-dark.svg

      - uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: output-svg-only
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  deploy-ghpages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          cache: yarn
          node-version: 16
      - run: yarn install --frozen-lockfile

      - run: yarn build:demo
        env:
          GITHUB_USER_CONTRIBUTION_API_ENDPOINT: https://snk-one.vercel.app/api/github-user-contribution/

      - uses: crazy-max/ghaction-github-pages@v2.6.0
        if: success() && github.ref == 'refs/heads/main'
        with:
          target_branch: gh-pages
          build_dir: packages/demo/dist
        env:
          GITHUB_TOKEN: ${{ secrets.MY_GITHUB_TOKEN_GH_PAGES }}

