# Deploying from circleci2.0 to Heroku
### Resources:
* [circleci 2.0 docs](https://circleci.com/docs/2.0/deployment_integrations/#heroku)
* [Managing SSH Keys on Heroku](https://devcenter.heroku.com/articles/keys)

### Steps:
Prerequisites:

  * Heroku CLI installed on your computer
  * A project deployed to Heroku
  * A project in circleci set up using circleci version 2.0

1. Edit `.circleci/config.yml`
  - Nested under `steps` (after you run tests, etc.) add:
  ```
  - run: bash .circleci/setup-heroku.sh
  - add_ssh_keys:
      fingerprints:
        - $HEROKU_SSH_FINGERPRINT
  - deploy:
      name: Deploy to Heroku if tests pass and branch is master
      command: |
        if [ "${CIRCLE_BRANCH}" == "master" ]; then
          git push --force git@heroku.com:$HEROKU_APP_NAME.git HEAD:refs/heads/master
          <any other necessary commands for heroku deployment you might need can go here>
        fi
  ```
2. Create a new file `.circleci/setup-heroku.sh` with the following:

  ```
  #!/bin/bash
  ssh-keyscan -H heroku.com >> ~/.ssh/known_hosts
  # If you need access to the heroku CLI to run heroku commands in the deploy step add these lines:
  # mkdir -p /usr/local/lib /usr/local/bin
  # tar -xvzf heroku-linux-amd64.tar.gz -C /usr/local/lib
  # ln -s /usr/local/lib/heroku/bin/heroku /usr/local/bin/heroku

  cat > ~/.netrc << EOF
  machine api.heroku.com
    login $HEROKU_LOGIN
    password $HEROKU_API_KEY
  EOF

  cat >> ~/.ssh/config << EOF
  VerifyHostKeyDNS yes
  StrictHostKeyChecking no
  EOF
  ```
3. add Environment Variables to circleci

Key Name | Where to get it
--- | ---
`HEROKU_API_KEY` | Found in Heroku Account Settings: It is your individual API Key even if it is a company Account
`HEROKU_LOGIN` | Your Heroku login e.g: `name.lastname@company.com`
`HEROKU_APP_NAME` | The name of your app on Heroku

4. Create SSH keys
  * Create a SSH key locally
  `$ ssh-keygen -t rsa`
  * Copy the fingerprint returned from ^^^^
  * Add another environment variable in circleci
    - HEROKU_SSH_FINGERPRINT
      - paste the copied fingerprint as the value
  * Add the public SSH to Heroku
  `$ heroku keys:add`
  * Open ~/.ssh/id_rsa (or wherever you saved your ssh key) and copy key including `-----Begin RSA PRIVATE KEY ----` and `-----End RSA PRIVATE KEY------`
  * Add SSH private key to circleci
    * in project settings -> SSH Permissions -> `Add SSH Key`
    * Hostname = git.heroku.com
    * Paste the private key and click `Add SSH Key`
    
5. Celebrate (trigger circleci on your master branch and watch it deploy!)

![celebrate](https://media.giphy.com/media/xT8qBmk4MAjBeTO1tm/giphy.gif)