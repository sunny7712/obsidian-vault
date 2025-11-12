#reference #tips 
1. Jackson errors. Check the involved classes properly. Check for any missing constructors in classes.
2. If you try everything and nothing works, try changing java 23 to 21.
	1. Works for lobmok annotation processing error.
	2. Works for mock beans.

3. create a local branch to track a origin branch
	`git checkout -b feature-branch origin/feature-branch`

4. In a multi module environment, you will have to add a module dependency to the server module's pom.xml for it to be available in the class path while component scanning.
5. You **created a local branch** and **pushed** it to the **origin** (remote). Meanwhile, **other commits** were added to the **origin branch** (by you or others). Now you have **new local changes** and want to **push them** to the origin branch.
```shell
git fetch origin
git rebase origin/your-branch
# resolve conflicts
git push origin your-branch
```

6. If you are getting any error in `GrowwApexBaseResponse` while deploying SDK, then check the java version. It must be 21.