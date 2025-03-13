# Deploy Site to to Pantheon (GitHub Action)


This GitHub Action deploys a site to Pantheon as a step in your GitHub Action workflow. It is meant to be used within individual repositories that contain the code a single site.


It is meant to be used in workflows that run upon pushes to Pull Requests and pushes to the `main` branch of a repository.


When running workflow triggered by a pull request, this action will create a [Multidev environment](https://docs.pantheon.io/guides/multidev) and deploy code to it.


TODO - Diagram


When running on workflows triggered by merges/pushes to the `main` branch this action will deploy code to [the Pantheon `Dev` environment](https://docs.pantheon.io/pantheon-workflow).


TODO - Diagram


## Basic Usage


This action provides a step that can be used as the only step within a job.
More complex examples further below show additional steps and jobs used in conjunction with this action.
Here is the beginning of a `jobs` section of [a real `.github/workflows/deploy-pr.yml` file](https://github.com/stevector/stevector-composed/blob/c93aef3a96f29054772486bafa9a3b805065003b/.github/workflows/deploy-pr.yml) that deploys a site to Pantheon when triggered by a Pull Request.


```
jobs:
 build:
   runs-on: ubuntu-latest
   steps:
   - name: Deploy to Pantheon
     uses: stevector-streaming/dtp@0.2.0
     with:
       ssh_key: ${{ secrets.PANTHEON_SSH_KEY }}
       machine_token: ${{ secrets.PANTHEON_MACHINE_TOKEN }}
       site: ${{ vars.PANTHEON_SITE }}
```






## Arguments




### Required Arguments


```yml
 ssh_key:
   description: 'A private key that corresponds to a public key on Pantheon: https://docs.pantheon.io/ssh-keys'
   required: true


 machine_token:
   description: "A token for interacting with Pantheon's command line: https://docs.pantheon.io/machine-tokens"
   required: true


 site:
   description: 'The machine name of your Pantheon site'
   required: true
```


### Optional Arguments


```yml
 target_env:
   description: 'The Pantheon environment to which the deployment will be made. If left blank, the value used will be automatically derived. Pull requests will deploy to environments named "pr-[NUMBER]" and main/master branch commits will deploy to the Pantheon "dev" environment'
   required: false
   default: ""


 source_env:
   description: "The environment from which the database and uploaded files will be copied."
   required: false
   default: "live"


 clone_content:
   description: "If set to true, the database and files directory will be re-cloned from the source environment. When set to false, this data is only copied upon Multidev creations. Setting this variable to true ensures fresh content but adds time to the build process that can be prohibitive for sites with large databases."
   required: false
   default: false
   type: boolean


 delete_old_environments:
   description: "If set to true, Multidev environments associated with closed pull requests will be deleted after deployment completes. It is recommended to set this parameter to true for workflows that run after merges to the main branch."
   required: false
   default: false
   type: boolean


 relative_site_root:
   description: "The root directory of the site to be deployed relative to the repository root. The vast majority of users of this action should leave this value unchanged from the default. The action will use this value to change directories after checking out the repo."
   required: false
   default: ""


 git_user_name:
   description: "The name to be used with the Git commit that will be pushed to Pantheon. This value is not used on newer 'eVCS' sites for which there is no Pantheon-provided Git Repo"
   required: false
   default: "GitHub Action Automation"


 git_user_email:
   description: "The email address to be used with the Git commit that will be pushed to Pantheon. This value is not used on newer 'eVCS' sites for which there is no Pantheon-provided Git Repo"
   required: false
   default: "GitHubAction@example.com"


 git_commit_message:
   description: "A custom commit message to be used with the Git commit that will be pushed to Pantheon. Leaving this Action parameter blank will result in a generic commit message being used. This value is not used on newer 'eVCS' sites for which there is no Pantheon-provided Git Repo"
   required: false
   default: ""
```


## Additional recommendations


### Additional build steps like `composer install` and `npm build`


_todo: explain_




### Concurrency


Sometimes in the course of development it is normal to push one commit to a branch with a pull request and then push another commit a minute later and then another. Similarly, a team might merge five pull requests in quick succession.
Depending on the nature of the project, the team might want relevant Workflows to be processed for every single commit.


However, for most WordPress and Drupal teams deploying to Pantheon we recommend running no more than one workflow on a branch at a time because this GitHub Action presumes (though does not strictly enforce) that all workflow runs on a given branch will deploy code to the same Pantheon environment.
Multiple workflows each attempting to deploy code to the same environment at the same time could result in confusing error states or failing automated tests that follow deployment.


To ensure that only one build runs at a time for a pull request, include this `concurrency` section in your workflow's yml file


```yml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number }}
  cancel-in-progress: false
```


For a workflow that handles only the main branch, that section could be altered to:


```yml
concurrency:
  group: ${{ github.workflow }}-main
  cancel-in-progress: false


### Testing Jobs


#### Unit Tests: run in parallel with the a `build` job


_todo: explain._


#### End-to-End: Run in sequence after a `build` job


_todo: explain._


