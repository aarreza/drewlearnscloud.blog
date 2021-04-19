---
layout: post
title:  "Deploy a Jekyll Blog to Azure Static Web Apps"
date:   2021-04-16 16:21:21 -0400
categories: [Software]
tags: web
permalink: /jekyll
---

As a follow up to my previous blog on [Installing Jekyll on OSX Big Sur](/web). I finally have a fully working local install of Jekyll, which I have also setup as a test environment for publishing my drafts and any site changes. I decided to deploy my [blog](/) using the following: 
- [Github](https://github.com/) - check out my repo [here](https://github.com/aarreza/drewlearnscloudblog)
- [Github Actions](https://docs.github.com/en/actions) 
- [Static Web Apps in Azure](https://azure.microsoft.com/en-ca/pricing/details/app-service/static/#:~:text=Static%20Web%20Apps%20is%20free,Azure%20Functions%20Consumption%20plan%20rates.)

All of these resources are **free** at the time of this post. You can set up an [Azure](https://azure.microsoft.com/en-us/free/) account for free for 12 months plus you get $260 USD worth of credits! Static Web App in Azure is currently on Preview until some time at the end of this year. The only costs I have so far is purchasing a domain name for a year for about _$10 CAD_ using [NameCheap](https://www.namecheap.com/). I also use [Cloudflare](https://www.cloudflare.com/plans/) for free as a Content Distribution Network and DNS. 

For a [full tutorial](https://docs.microsoft.com/en-us/azure/static-web-apps/publish-jekyll) on how to deploy a test Jekyll site, check out this post from Microsoft. 

## **Create a Jekyll Site**
Create a Jekyll site using Command Line Interface (CLI) - _Bash_ or _Zsh_ Terminal. 

1. I already have a Jekyll site called `jekyll-first-website` from our previous [post](/web). Navigate to the newly created app directory. You should have a `Gemfile`. 

{% highlight console %}
cd jekyll-first-website
{% endhighlight %}

2. Initialize a new Git repository.

{% highlight console %}
git init
{% endhighlight %}

3. Commit the changes

{% highlight console %}
git add -A
git commit -m "Initial commit"
{% endhighlight %}

## **Setup a GitHub repository**
1. Create a blank GitHub repo from [https://github.com/new](https://github.com/new). 
2. Add the GitHub repository as a remote to your local repo. Change `<YOUR_USER_NAME>` placeholder with your GitHub username and `<GITHUB_REPO>` with the repo name you created in Step 1.

{% highlight console %}
git remote add origin https://github.com/<YOUR_USER_NAME>/<GITHUB_REPO>
{% endhighlight %}

3. Push your local repo up to Github. Ensure that you are on the root directory of your Jekyll site.

{% highlight console %}
git push -u origin master
{% endhighlight %}

## **Deploy your site to Azure**

1. Navigate to the [Azure Portal](https://portal.azure.com/).

2. Click the '+' sign **Create a Resource**. 

3. On the Search bar look for *Static Web App (preview)*.

4. Click _Create_. 

5. For _Subscription_, accept the default. I am using _Free Trial_ at this time.

6. In _Resource Group_, select **New**. Enter `jekyll-first-website` and select **OK**.

7. Under _Static Web App details_. Enter `jekyll-first-website` in the Name box. 

8. For _Region_, select an available [Azure region](https://azure.microsoft.com/en-ca/global-infrastructure/geographies/) close to you.

9. For _SKU_, select **Free**.

![Image](/assets/images/azurestatic.png)

10. Click the **Sign in with GitHub** button.

11. Click the **Authorize Azure-App-Service-Static-Web-Apps**.

12. Select the Account or Organization under which you created the repo. In our case it will be `jekyll-first-website`. 

13. For the _Branch_, select `master`.

14. Click on **Next: Tags >** button and choose the defaults.

## **Build**

1. Click the **Next: Build >** button to edit the build configurations.

2. Set _App location_ to `/_site`.

3. Leave the _App artifact location_ blank.

## **Review and Create**

1. Click the **Review + Create** button and verify all the details.

2. Click **Create** to start the creation of Azure Static Web App and provisioning of a Github Action for deployment.

## **Edit the YAML file in Github**

1. The new deployment will fail at first run as the default workflow file needs to be edited for Jekyll. 

2. Go back to your Jekyll site and terminal. This command will fetch and download the newly added content from your remote repository.

{% highlight console %}
git pull
{% endhighlight %}

3. Open the file name in `.github/workflows/<WORKFLOW_NAME>.yml` file. You can reference mine [here](https://github.com/aarreza/drewlearnscloudblog/blob/master/.github/workflows/azure-static-web-apps-proud-forest-0b3b9590f.yml). 

4. Add the following configuration block before `-name: Build And Deploy`. This code checks out a private repository with `uses: actions/setup-ruby@v1` and in my case my local Ruby installed version is `3.0`. This will automatically install all the dependencies requires.

{% highlight yml %}
 - name: Set up Ruby
        uses: actions/setup-ruby@v1
        with:
          ruby-version: 3.0
      - name: Install dependencies
        run: bundle install
      - name: Jekyll build
        run: jekyll build
{% endhighlight %}

5. Commit the updated workflow YAML file and push to Github. 

{% highlight console %}
git add -A
git commit -m "Updating GitHub Actions workflow"
git push
{% endhighlight %}

6. By entering `git push`, GitHub Actions will trigger the build for your site. The first time creating your site, it will take a while. 

![Image](/assets/images/githubactions.png)

7. Once GitHub Actions completes. Go back to the Azure Static Web App. Under _Overview_, click on the _URL_ to view your deployed site. 

At this, point my site is now up and running with a default _URL_ assigned by Azure. I will be setting up my `drewlearnscloud.blog` custom domain at a later post. 