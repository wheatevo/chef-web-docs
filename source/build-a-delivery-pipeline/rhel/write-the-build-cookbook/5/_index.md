## 5. Run smoke tests and deploy to your Union, Rehearsal, and Delivered stages

In the previous step, you manually verified that the Customers application is running in Acceptance. In practice, it's common to have some manual verification process to validate that your application or service is up and functioning. For example, if you're deploying a web application, someone will typically test out a new feature manually on a pre-production server before releasing the feature to production.

However, you can also write _smoke tests_ to help quickly validate that the application or service is running and functional. If the smoke tests fail, you know that the application or service has failed.

### Create a branch

First, verify that you're on the `master` branch.

```bash
# ~/Development/deliver-customers-rhel
$ git branch
  add-delivery-config
  add-delivery-truck
  deploy-customers-app
* master
  provision-environments
  publish-customers-app
```

Run these commands to create the `smoke-test-customers-app` branch and verify that you're on that branch.

```bash
# ~/Development/deliver-customers-rhel
$ git checkout -b smoke-test-customers-app
Switched to a new branch 'smoke-test-customers-app'
$ git branch
  add-delivery-config
  add-delivery-truck
  deploy-customers-app
  master
  provision-environments
  publish-customers-app
* smoke-test-customers-app
```

### Write the smoke recipe

Smoke tests are meant to be fast so that you quickly receive feedback if the application or service is not working. For the Customers web application, we'll simply run cURL to verify that the home page comes up and that the server responds with 200 (OK) HTTP status code.

Write your `smoke` recipe like this.

```ruby
# ~/Development/deliver-customers-rhel/.delivery/build-cookbook/recipes/smoke.rb
include_recipe 'delivery-truck::smoke'

# Create a search query that matches the current environment.
search_query = "chef_environment:#{delivery_environment}"

# Run the query.
nodes = delivery_chef_server_search(:node, search_query)

# cURL the IP address of each result and verify a 200 (OK) response.
nodes.each do |node|
  address = node['ipaddress']
  execute "cURL #{address} and verify 200 response" do
    command "curl -IL #{address} | grep '^HTTP/1\.1 200 OK'"
  end
end
```

This code performs a similar query as the deploy phase. For each node in the environment (we expect only one), we use the `execute` resource to run `curl` with the `-IL` flag and search for the expected response code.

### Review and approve the change

Let's try out our smoke test. Follow the same steps as before to submit your change and trigger the pipeline.

```bash
~/Development/deliver-customers-rhel
$ git status
On branch smoke-test-customers-app
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   .delivery/build-cookbook/recipes/smoke.rb

no changes added to commit (use "git add" and/or "git commit -a")
$ git add .
$ git commit -m "smoke test environments"
[smoke-test-customers-app 1b7b529] smoke test environments
 1 file changed, 14 insertions(+)
$ delivery review
Chef Delivery
Loading configuration from /home/thomaspetchel/Development/deliver-customers-rhel
Review for change smoke-test-customers-app targeted for pipeline master
Created new patchset
https://10.194.11.99/e/test/#/organizations/learn-chef/projects/deliver-customers-rhel/changes/639d2844-c94a-4015-b4cb-bc000b1c9172
```

Trace the change's progress through the pipeline to the Acceptance stage.

1. Review the changes in the web interface. Click **Approve** when all tests pass.
1. Watch the change progress through the Build and Acceptance stages.

You'll see that the smoke test passes.

![](delivery/acceptance-smoke-test.png)

Previously, you verified your change up to the Acceptance stage. This time, after Acceptance succeeds, press the **Deliver** button, then press **Confirm**.

![](delivery/deliver-customers.png)

This moves the change through the Union, Rehearsal, and Delivered stages.

![](delivery/customers-delivered.png)

### Verify the change was successfully delivered

* Let's verify that the `awesome_customers` cookbook successfully deployed to your Delivered stage.

* As before, you'll need the IP address of your server.
* If you're using the SSH driver, you already have it.
* If you're using the AWS driver, you can get its IP address from its node attributes.

If you're using the AWS driver, move to the <code class="file-path">~/Development/delivery-cluster/.chef</code> directory.

```bash
# ~/Development/deliver-customers-rhel
$ cd ~/Development/delivery-cluster/.chef
```

Now run `knife node show`, providing the name of the node for your Delivered stage, and then search for the node's IP address.

```bash
# ~/Development/delivery-cluster/.chef
$ knife node show delivered-deliver-customers-rhel-aws | grep IP:
IP:          10.194.15.90
```

Now move back to your <code class="file-path">~/Development/deliver-customers-rhel</code> directory.

```bash
# ~/Development/delivery-cluster/.chef
$ cd ~/Development/deliver-customers-rhel
```

From a web browser, navigate to your node's IP address.

![](delivery/delivered-customers-verify.png)

Congratulations! You've successfully delivered the Customers web application!

### Integrate the change locally

As we did previously, let's merge the `master` branch locally. Here's a reminder how.

```bash
# ~/Development/deliver-customers-rhel
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'delivery/master'.
$ git fetch
remote: Counting objects: 1, done.
remote: Total 1 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (1/1), done.
From ssh://test@10.194.11.99:8989/test/learn-chef/deliver-customers-rhel
   ee4ff54..e46f7b2  master     -> delivery/master
$ git pull delivery master
From ssh://test@10.194.11.99:8989/test/learn-chef/deliver-customers-rhel
 * branch            master     -> FETCH_HEAD
Updating ee4ff54..e46f7b2
Fast-forward
 .delivery/build-cookbook/recipes/smoke.rb | 14 ++++++++++++++
 1 file changed, 14 insertions(+)
```

[GITHUB] The final code for this section is available on [GitHub](https://github.com/learn-chef/deliver-customers-rhel/tree/smoke-test-customers-app-v1.0.0) (tag `smoke-test-customers-app-v1.0.0`.)