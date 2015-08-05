## 3. Add rules that trigger an alert when an audit fails

In [Get started with Chef Analytics](/get-started-with-chef-analytics/linux/), you created an email notification that is sent in response to a condition, or rule. In this tutorial, instead of using a notification to respond to an event, you'll create an alert. An _alert_ enables you to capture important events that you want to take action on, such as when an audit fails.

You access the alert history through the Chef Analytics web interface. Like a notification, an alert is raised from a rule. You can generate an alert and a notification from the same rule.

Let's write a rule that's triggered when your audit control, which verifies that web content must not be owned by the `root` user, fails.

Recall that your control looks like this.

```ruby
# ~/chef-repo/cookbooks/audit/recipes/default.rb
control_group 'Validate web services' do
  control 'Ensure no web files are owned by the Administrators group' do
    Dir.glob('c:/inetpub/wwwroot/**/*.htm') {|web_file|
      it "#{web_file} must not be owned by Administrators" do
        expect(command("(Get-ChildItem #{web_file} | Get-Acl).Owner").stdout).to_not match(/Administrators$/)
      end
    }
  end
end
```

To create a rule that triggers when this audit fails, first navigate to the Chef Analytics interface from your web browser. From the **Rules** tab, click **+** to create a new rule. From the rule editor, click `New Rule Group 1` and rename it to `Validate web services`.

Now add the following code to define your rule.

```ruby
rules 'Validate web services'
  rule on run_control
  when
    name =~ 'is not owned by Administrators$' and status != 'success'
  then
    alert:error('Run control group "{{ message.name }}" failed on {{ message.run.node_name }}.')
  end
end
```

The [run_control](https://docs.chef.io/analytics_rules.html#run-control) message states a rule for a single audit. The `name` part of the `when` block corresponds to the name of the `it` block in your audit control. The rule triggers only when the status of the control is not `success`.

The `it` part of your control has a different name for each file that you're testing. Therefore, we use a [regular expression](https://docs.chef.io/analytics_rules.html#regular-expressions) to match the pattern for how the `it` blocks are named. The `=~` operator sets up the comparison as a regular expression and the `Administrators$` part says that the string to match must end with "Administrators".

[TIP] [Rabular](http://rubular.com) is a handy way to test out your regular expressions.

The [alert:error](https://docs.chef.io/analytics_rules.html#alert-level) part adds the alert to Chef Analytics. In our case, we want the alert to signal an error condition. You can also use `alert:warn` and `alert:info` to signal other types of conditions.

Click **Save** and you are brought back to the list of rules.