# GithubAutoBackupsFramework

#Introduction:

As a QA tester, I often find myself navigating the complex landscape of collaborative development. With team members pushing and pulling code every day, the need for regular and reliable backups becomes paramount. After all, the last thing we want is for a single misplaced line of code to throw the entire project into chaos. Inspired by this challenge, I developed a model to automate backups using GitHub Actions, and I'd love to share it with you all.

#Why Automate Backups?

Imagine this: you're working on a crucial feature, and suddenly, someone else’s changes break the main branch. Panic ensues, and you’re left wondering why you didn’t back up more often. Sound familiar? That’s where automation comes in, ensuring peace of mind and a safeguard against those “Oops, I did it again” moments.

#The Solution: My GitHub Action Workflow

Here's a peek into the code that powers this automation:

bash
Copy code
name: Backup Repository

on:
  schedule:
    - cron: '0 * * * *'  # Runs every hour
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Fetch all branches
          token: ${{ secrets.PAT }}

      - name: Set up Git
        run: |
          echo "Setting up Git configuration"
          git config --global user.name 'github-actions'
          git config --global user.email 'github-actions@github.com'

      - name: Create backup branch
        id: create_branch
        run: |
          echo "Creating backup branch"
          TIMESTAMP=$(TZ="Europe/Istanbul" date +"%Y-%m-%d-%H-%M")
          BRANCH_NAME="backup-$TIMESTAMP"
          echo "Branch name: $BRANCH_NAME"
          git checkout -b $BRANCH_NAME
          echo "BRANCH_NAME=$BRANCH_NAME" >> $GITHUB_ENV

      - name: Add backup remote
        run: |
          echo "Adding remote backup repository"
          git remote add backup https://${{ secrets.PAT }}@github.com/kn-oz/backupfilesAPITEAM3.git
          git remote -v  # List remotes to verify

      - name: Pull changes from backup repository
        run: |
          echo "Pulling changes from backup repository"
          git pull backup main --rebase || echo "No changes to pull"

      - name: Check PAT permissions
        run: |
          echo "Checking PAT permissions"
          curl -H "Authorization: token ${{ secrets.PAT }}" https://api.github.com/user/repos

      - name: Push to backup repository
        run: |
          echo "Pushing to remote repository"
          git push backup ${{ env.BRANCH_NAME }} --force
##How It Works

Create a Backup Repository: Start by creating a separate repository that will serve as your backup storage. This repository will hold all your backup branches.
Set Up GitHub Actions: Add the above code to the GitHub Actions workflow in your source project. This configuration runs every hour, ensuring regular backups.
Generate a Personal Access Token (PAT): In your GitHub account, generate a PAT with appropriate permissions (repo access). This token allows the GitHub Actions workflow to authenticate and perform operations on your repositories.
Add PAT to Secrets: In your source project repository, navigate to Settings > Secrets, and add your PAT as a secret (e.g., PAT). This ensures secure access by the GitHub Actions workflow.
Workflow Steps:
Checkout Repository: The workflow checks out your source repository.
Set Up Git: It configures Git with a username and email for the GitHub Actions bot.
Create Backup Branch: A new branch named with the current date and time is created.
Add Backup Remote: The backup repository is added as a remote named backup.
Pull Changes: Changes are pulled from the backup repository to keep it up to date.
Check PAT Permissions: The workflow verifies that the PAT has the necessary permissions.
Push to Backup Repository: The new backup branch is pushed to the backup repository.
The Benefits

#Peace of Mind: Automated backups mean you can focus on coding, knowing that your work is safe.
Quick Recovery: In case of any issues, restoring from a recent backup is straightforward.
Version History: Each backup branch serves as a historical record, useful for auditing changes over time.
#Wrapping Up

Automating GitHub backups has been a game-changer for our team. It’s like having a safety net that’s always there, ready to catch us when we fall. Plus, it’s a great way to avoid those late-night “code broke, need fix” nightmares.

If you found this approach interesting or think it could help your team, I’d love to hear your thoughts. Feel free to reach out or drop a comment. Let's keep our code safe and our minds at ease!

P.S. If you’re curious to implement this in your project or have any questions, don't hesitate to get in touch. Happy coding!
