![gitops](/resources/gitops.png)
## What is GitOps?
### GitOps was originally defined by [WeaveWorks](https://www.weave.works/technologies/gitops/) as a way to have Git at the center of your delivery pipeline. After implementing GitOps, Git becomes the source of truth for deployment and delivery activities of your application and infrastructure.
### While the original purpose of GitOps was to manage deployments and tasks for Kubernetes, we can use this approach to improve Continuous Delivery practices for any application or change in your delivery pipeline.

## How does this work?
I have created this repository to help tell the story of how you could use GitOps for your application deployments.

This pattern should not be specific to a specific technology or application stack but defines a "GitOps"-like process to promote your application through environments that correspond to branches in a git repository.

You could extend this to any "as-code" deployment for any application or infrastructure change.

This repository contains the following files:
* [manifest.json](manifest.json): A versioned, source-controlled file that describes which version of the application should be deployed to any environment.
* [Jenkinsfile](Jenkinsfile): A pipeline script to read in the manifest.json and parse environment variables such as your application version and artifact location. 
* [dev-env.json](dev-env.json), [staging-env.json](staging-env.json), [prod-env.json](prod-env.json): Files that describe the environment and deployment information for your applications.  These files could be replaced by results from API calls to configuration management or other methods to provide your deliver system with endpoints for deployment or testing.  

To summarize this process, you will be creating a manifest that has application data such as a specific version or application type to help define what will be deployed.  This manifest can have one or many applications listed and can be used as a representation of a stack or "environment" of connected products.  Each of the applications you need in a product release could be represented here.  To promote the application or stack through environments, you will be "pulling" the application through its CD pipeline by creating Pull Requests of a manifest to Git branches that correspond to environments in the application's path to production.

By using this process, you are able to define exactly which version of your application(s) and which deployment steps are defined **in code** so that it is always clear which applications exist in which environment and you can control who has the ability to deploy to specific environments by simply having certain team members manage PR merges to specific branches. 

Treating everything as code is super important and GitOps allows us to take this practice to deploying any of your applications or infrastructure code.

Many CI/CD tools could be used to accomplish this functionality.  The basic requirements are to be able to parse a manifest and perform certain tasks based on git branch-specific logic.

I have chosen to use a Jenkinsfile (declarative Jenkins pipeline) to illustrate the process as follows:
### Preparation
1. Create a starter manifest in a "master" branch to outline the information needed to deploy any artifact to any location.
   * Refer to [manifest.json](manifest.json) as an example. I started with *dev* but you can look at it as the lowest environment not deployed to automatically by your CI process.
   * You should pay attention to the `app_version` as this is the most important item.  This along with the `manifest_version` is the section that would change most often.  They may be different because you may choose to deploy multiple artifacts/versions of different applications.
   * Remember, this process is for deploying anything anywhere or promoting from env to env but we want to make sure we're using versioned artifacts.
2. Prior to deploying your application to any environments, you will define your application's "path to production" by creating a branch per environment in your "GitOps deployment repo". Create one branch per environment defined in your app's path to prod (like **dev -> staging -> production**). Use this repo as an example.
    * Note that this is independent of your actual "_code_" repository. We're not talking about the application code here but instead deploying the immutable artifacts or infrastructure it creates.
    * This manifest/gitops repo may be completely disconnected from your application builds but could also be updated by CI to update a version if you like. *This is for deployment processes only.*
3. Create a deployment script like the [Jenkinsfile](Jenkinsfile) to outline the specific stages and parses values from the manifest corresponding to the branch logic applied to each environment.
   * Feel free to use any CI tool/process you like here. The declarative Jenkinsfile makes it super simple to enable branch-specific logic we can use for GitOps
   * You will need to script methods to pull your artifacts from artifact storage locations such as Nexus or Artifactory.  Those methods aren't defined here because they will be specific to your implementation.


### Execution
1. The deployment of your application can be triggered in one of two ways: Automated "Continuous Delivery" system updates or via a Pull Request
   1. The CI system can update the manifest in the **dev** branch
      *  After a certain amount validation, TBD by your team, the automated system can update the manifest with the latest version of your application
      *  This is important if you wish to support **Continuous Delivery** or **Continuous Deployment**
   2. You can initiate deployment to your **dev** environment by updating your manifest and creating a PR (Pull Request) to the Environment of your choice.
      * Normally you'll PR from environment to environment to **promote** your application but you can PR from any branch to any branch. What if you want to take what's in **production** and deploy it to **qa**? The main point here is to make any change traceable
2. If deploying/promoting your artifacts manually by PR, you will assign reviewers to help validate the correct application version in the manifest or any other changes that need review.
3. Depending on your process, you can merge into the environment yourself after approvals or allow others the honor of merging into target environments.
   * 
4. Upon PR merge (or automated commit), the process will kick off and deploy the application listed in the manifest to the environment aligned to the branch you're merging into. 
   * Dev deployment: ![dev deployment](/resources/dev_deploy.png) 
   * Staging deployment: ![staging deployment](/resources/staging_deploy.png)
   * Production deployment: ![production deployment](/resources/production_deploy.png)

## Restricting Deployment to Production or Protected Environments
 * You may wish to make the production depployment more restrictive
   * Merge restrictions come into play in an enterprise environment where you may wish to assign certain team members the ability to deploy to some environments but not others
   * You may want to restrict Production deployments to a certain team.  You can lock down merge capabilities to some branches/environments and require certain reviewers in others 
      * GitHub, among others, enable users to control these restrictions at a very granular level:
      * ![merge restrictions](/resources/merge_restrictions.png)
* **NOTE: You can lock this process down further or for production deployments by creating another repository where a separate build system that only deploys to production has access**.
    * This will enable you to have a protected set of credentials or an entire deployment system dedicated to production deployments.

### Summary/Conclusion
We will always need a way to be certain which applications are deployed in which environments/regions.  It doesn't matter if you create your environments dynamically, run them in Kubernetes, or just deploy to the same static environments with each release. 

This is currently just an example of how this could be implemented in your environment.  You can reimagine this with more or less complicated manifests and deployment processes.  I will be updating this to be a bit more functional but welcome your comments and improvements.
