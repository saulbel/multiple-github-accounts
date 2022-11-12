# multiple-github-accounts

## Prerequisites
Things you need before starting:
* `Multiple GitHub accounts`
* `Linux/Unix OS`

## Tasks to accomplish
- Let's say that we work with our own personal pc or laptop and we want to have access to both, our personal `GitHub` account and the work one. We are gonna explain how to achieve this.

## First step: create ssh keys
- First of all we are create the ssh keys for both GitHub accounts:
````
$ cd /home/your_user/.ssh/
$ ssh-keygen -t rsa -b 4096 -C "your_personal_email@domain.com"
# we can choose the name of the file our ssh key will be stored, on my case github_personal
$ ssh-keygen -t rsa -b 4096 -C "your_work_email@domain.com"
# we can choose the name of the file our ssh key will be stored, on my case github_work
````
- Those commands above will generate the following files: `github_personal`, `github_personal.pub`, `github_work` and `github_work.pub`

## Second step: add ssh keys to GitHub
- Now we are gonna upload both `.pub` keys to our GitHub accounts. How do we do this?
- First, we are gonna copy the output of our `github_personal.pub` to our clipboard:
````
$ cat /home/your_user/.ssh/github_personal.pub
````
- Second, we will log in on our GitHub account and then we will go to `Settings > SSH and GPG Keys > New SSH key` and there we will paste our `github_personal.pub` key.
- Repeat both steps for `github_work.pub`.

## Third step: setup configuration files 
- First we are gonna create a `github_config` file in our user `.ssh` folder and we will include the following info (you do not have to modify it at all):
````
$ tee /home/your_user/.ssh/github_config <<EOF
# Personal account 
Host github.com-personal
   HostName github.com
   User git
   IdentityFile ~/.ssh/github_personal
# Work account
Host github.com-work
   HostName github.com
   User git
   IdentityFile ~/.ssh/github_work
EOF
````
- Second we are gonna create a `.gitconfig` file in our user root directory (replace name and email):
````
$ tee /home/your_user/ <<EOF
[user]
    name = your_user_github_name
    email = your_work_github_email
[includeIf "gitdir:~/work/"]
    path = /home/your_user/work/.gitconfig
EOF
````
- Third, now we are gonna create a couple of directories in our user root directory. The idea of this setup is that each directory will be linked to a GitHub account.
````
$ mkdir personal
$ mkdir work
````
- Now inside of each directories we are gonna create a `.gitconfig` file:
````
$ tee /home/your_user/personal/.gitconfig <<EOF
[url "ssh://git@github.com/"]
[user]
    email = your_personal_email@domain.com
EOF

$ tee /home/your_user/work/.gitconfig <<EOF
[url "ssh://git@github.com/"]
[user]
    email = your_work_email@domain.com
EOF
````
## Fourth step: add ssh keys to ssh-agent
- First we are gonna start the `ssh-agent`
````
$ eval "$(ssh-agent -s)"
````
- Then we are gonna add both keys:
````
$ ssh-add /home/your_user/.ssh/github_personal
$ ssh-add /home/your_user/.ssh/github_work
````
- You may want to add this to your `.bashrc` in order to initialize the `ssh-agent` each time you log in.
````
$ eval `ssh-agent -s` > /dev/null 2>&1
````
- We can check if our keys have been added succesfully:
````
$ ssh-add -l
````
## Fifth step: test it on a GitHub repo
````
$ git clone git@github.com:saulbel/python-shopping-basket.git
$ cd python-shopping-basket
$ git checkout -b test-branch
$ tee test.txt <<EOF
this is a test
EOF
$ git add test.txt
$ git commit -m "testing GitHub repo"
$ git push origin test-branch
````

