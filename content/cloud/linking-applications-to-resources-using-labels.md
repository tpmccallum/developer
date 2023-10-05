title = "Linking Applications to Resources Using Labels"
template = "cloud_main"
date = "2023-10-30T00:22:56Z"
enable_shortcodes = true
[extra]
url = "https://github.com/fermyon/developer/blob/main/content/cloud/linking-applications-to-resources-using-labels.md"

---

- [Benefits of Links and Labels](#benefits-of-links-and-labels)
- [Working With Links and Labels](#working-with-links-and-labels)
- [Next Steps](#next-steps)

Spin applications are inherently ephemeral resources; therefore, the state of an application needs to be persisted in external stores such as NoOps SQL Databases. To do this, Spin applications need a way to refer to these external resources. As your workload grows into a distributed system, you will likely have multiple applications connecting to the same NoOps SQL Database with different names as they don't necessarily know or need to care about the terms used by the other.

This is where Fermyon Cloud **links** and **labels** are helpful. As a Spin application developer, you provide a label whenever you fill out the name for a NoOps SQL database in the component manifest. Let's walk through a specific example together:

```toml
// --snip--
name = "todo-app"
[component]
id = "todo"
sqlite_databases = ["data"]
// --snip--
```

The string "data" is a **label** that will be used by the Spin application to reference the NoOps SQL Database. At application deployment time, we get a couple of choices from the `spin` command:
- Use an existing database and link app to it
- Create a new database and link the app to it

In this example below, we are going to **link** the label to a specific NoOps SQL Database instance. To do this, we choose the first option (that `spin deploy` gives us). We make our selection by using the up/down arrow keys and then `return` to continue:

```bash
spin deploy
Uploading todo-app version 0.1.0-r234fe5a4 to Fermyon Cloud...
Deploying...
App "todo-app" accesses a database labeled "data"
Would you like to link an existing database or create a new database?:
> Use an existing database and link app to it
  Create a new database and link the app to it
```

At this point the `spin deploy` command asks us which database we would like to link to our `todo-app` using the label "data" (which we configured in our `spin.toml` file above):

```bash
Which database would you like to link to todo-app using the label "data":
> inspiring-otter
```

> Note: The database names are arbitrary and created on our behalf. In this case `inspiring-otter` is our Cloud database. We are linking the `inspiring-otter` database to our `todo-app` using the label "data".

From this point, we see that our app is deploying and once finished we receive our deployed application's URL endpoint:

```bash
Waiting for application to become ready...... ready
Available Routes:
  todo-app: https://todo-app-ylnmlqmr.fermyon.app (wildcard)
```

Let's kick things up a notch and deploy a second Spin application (`todo-app-v2`) that will also use the same Cloud database:

```toml
// --snip--
name = "todo-app-v2"
[component]
id = "todo"
sqlite_databases = ["default"]
// --snip--
```

When deploying the second Spin application (`todo-app-v2`) we are again asked if we would like to use an existing database and link our application (in this case `todo-app-v2`) to it:

```bash
spin deploy       
Uploading todo-app-v2 version 0.1.0-r6e577b18 to Fermyon Cloud...
Deploying...
App "todo-app-v2" accesses a database labeled "default"
Would you like to link an existing database or create a new database?:
> Use an existing database and link app to it
  Create a new database and link the app to it
```

We again use the up/down arrow keys to make our selection and hit `return` to continue:

```bash
Which database would you like to link to todo-app-mkii using the label "default":
> inspiring-otter
```

In the example here we have chosen to link `todo-app-v2` to our pre-existing `inspiring-otter` Cloud database using via the "default" label, that we added to our `todo-app-v2` application manifest (`spin.toml`) file.

Now we have two seperate Spin applications (`todo-app` and `todo-app-v2`) using their own labels to link to the same database. 

Links and labels are application scoped. Whenever a specific Spin application (say `todo-app`) needs to access the database resource (`inspiring-otter`), it will do so via the label "data".

The Spin application does not need to know the true name of the Cloud resource (`inspiring-otter` database) that it is interacting with. 

A label is understood by all the components within a single application, but the labels applied do not hold any significance to other Spin applications. 

> At this time, only NoOps SQL DB supports labels. However, in the future other Fermyon Cloud resources such as Key Value Store will support links and labels as well. 

## Benefits of Links and Labels

Benefits of the Fermyon Clould links and labels include:

* **Easy Sharing**: Share your Fermyon Cloud resource across applications effortlessly.
* **Resource Creation Control**: You decide when to create and delete Fermyon Cloud resources.
* **Flexible Lifecycle Management**: Manage Fermyon Cloud resources independently of your Spin application.
* **Seamless Cloud Integration**: Cloud experience smoothly integrates with local Spin development, no runtime configuration changes required. 
* **Dynamic App Resource Configuration**: Change the data resource your application points to without rebuilding it.

## Working With Links and Labels

Whenever you add a string to your NoOps SQL database entry in the component manifest, you're creating a label. When you deploy your Spin application to Fermyon Cloud, you will be prompted to link the label to a specific NoOps SQL database instance. Later on, if you'd like to update the link you can do so via the `spin cloud link` command. 

To see your existing labels, you will need to use the `spin cloud` plugin along with the subcommand of the resource you've linked your Spin application to (via a label). For example, if you would like to see the labels you created for your NoOps SQL DB resources, you would run the following command:

```bash
$ spin cloud sqlite list
+--------------------------------------------------------+
| App                          Label     Database        |
+========================================================+
| todo-app                     data      inspiring-otter |
| todo-app-v2                  default   inspiring-otter |
+--------------------------------------------------------+
```

Now you can see your Spin applications, their respective labels and their connected NoOps SQL Databases.

If you'd like to change your link while your Spin application is running, you can do so with the following command:

```bash
spin cloud sqlite unlink --app todo-app data
```

Now we've successfully unlinked our `todo-app` from the `inspiring-otter` database. 

If you choose to delete your Spin application without unlinking, you are deleting the link and label as well. This act has no impact on the NoOps SQL Database that was previously linked to the Spin application.

## Next Steps

* Review the [NoOps SQL DB Tutorial](noops-sql-db.md) to learn how to use links and labels with your NoOps SQL DB
* Visit the [Spin Cloud Plugin](cloud-command-reference.md) reference article to learn more about the `spin cloud sqlite link` commands