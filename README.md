# Practice Git Security
This repo is a quick tutorial on how to scan a repo for secrets, remove them from history, and prevent leaking of secrets. Go through these steps chronologically.

## Git Leaks

1. Fork this repo. If you simply clone instead, you won't be able to push the changes you make.
2. We're going to scan our repo with `gitLeaks` (though there are [many options](https://spectralops.io/blog/top-9-git-secret-scanning-tools/)). Follow the directions [here](https://github.com/zricethezav/gitleaks) to install it.
3. Make sure that you follow the instructions to install `pre-commit` (`pip install pre-commit`), and run `pre-commit autoupdate` and `pre-commit install`
4. Think about what we want to catch. GitLeaks has an impressive list of [rules](https://github.com/zricethezav/gitleaks/blob/master/config/gitleaks.toml) that it uses to identify data it strongly suspects are secrets. It will catch secrets that follow the format of common secrets like AWS secret keys and GCP API keys, or even obscure secrets like "Etsy Access Tokens." Is there something that we want to add? (no action)
5. We will add one rule, to catch when a file contains the exact string `PASSWORD=` followed by anything. At the root of this repo, create a file called `.gitleaks.toml` with the following content:
```
title = "Gitleaks Custom Rules"

[extend]
useDefault = true

[[rules]]
id = "exclude PASSWORD="
description = "checks for instances of PASSWORD= followed by digits"
regex = '''PASSWORD=[0-9]{0,20}'''

[allowlist]
description = "allow README, for examples"
paths = [
    '''README.md'''
]
```

6. Now, with our default and custom rules set up, let's test that we can't commit something bad. Create a file called `password.txt` with the following content:
```AWS_KEY=AKIAIMNOJVGFDXXXE4OA```
7. Add the file `git add password.txt` and commit it `git commit -m "uh oh, secret here"`. If you get an error, congratulations! You've blocked yourself from committing a secret!
8. Yeah, let's just delete that `password.txt`. We don't want that anymore.
9. Let's direct our attention to the repo and our history. If you look at our current files, we don't see anything that raises concern. It's just this `README.md`, `.pre-commit-config.yaml`, a `docs` folder with images, and a `.gitleaks.toml` you created. But anything can happen in our history. (no action)
10. Run `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"` (You can also run `git log`, but this will just be longer output, and we want something short and pretty). It should look something like the following, but longer:
```
cdf02c1 Nathaniel Larson        Fri Oct 28 03:01:33 2022 -0500  steps 5-9 added
6e82a4a Nathaniel Larson        Fri Oct 28 02:59:47 2022 -0500  steps 2-4 added
d18cd01 Nathaniel Larson        Fri Oct 28 02:48:13 2022 -0500  not suspicious
67f13d5 Nathaniel Larson        Fri Oct 28 00:34:54 2022 -0500  step 1 added
ca2dbf1 Nathaniel Larson        Fri Oct 28 00:33:55 2022 -0500  init README.md
```
11. Nothing suspicious there, right? Just kidding. One of those commits may contain a secret. We will use the `detect` command: `gitleaks detect`. It should tell us:
```
WRN leaks found: 1
```
12. Re-run, this time with the `-v` flag for a verbose output: `gitleaks detect -v`
13. Did you get an error for the `bad.env` file? It has broken our custom rule, as well as one of the default rules--but it is useful to note we only see the first (custom) rule listed in the warning. Expected output should look something like the following:
![Error using gitleaks](docs/error_gitleaks.png)
14. On to removing that secret that some bumbling idiot committed!!
