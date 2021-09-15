# Red Hat CodeReady Workspaces Workshop

:::warning
Credentials will be assigned during the workshop
:::

## Environment Access

You will be using [Red Hat CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview),
an online IDE. Changes to files are auto-saved every few seconds,
so you don't need to explicitly save changes.

To get started, access the CodeReady Workspaces Dashboard and
log in using the username and password you've been assigned.

If this is the first time logging in with an account, CodeReady Workspaces will request the user to authorize access to the account, click on the `Allow selected permissions` button to continue.

![Authorize Account Access](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-04.png)

| Name | URL |
| -------- | -------- |
| OCP Console | <https://console-openshift-console.apps.cluster-40fb.40fb.sandbox1485.opentlc.com> |
| CodeReady Workspaces Dashboard | <https://codeready-openshift-workspaces.apps.cluster-40fb.40fb.sandbox1485.opentlc.com/dashboard/#/workspaces> |

## Creating a new Workspace

Once you log in, you'll be placed on your personal dashboard.

![CRW Landing Page](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-01.png)

Click on the `Create Workspace` item on the left menu

![Create new Workspace](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-02.png)

You will create a workspace from an existing repository from GitHub.
In the `Git Repo URL` text field, enter the following [Devfile](https://redhat-developer.github.io/devfile/) URL and click the `Create & Open` button.

```
https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/devfile.yaml
```

After a couple of minutes a new Workspace will be created and the IDE will be displayed.

![Create new Workspace](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-05.gif)

### Identify Devfile elements in the Workspace

The [Devfile](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/devfile.yaml) used to create the Workspace has multiple elements that define the configuration and behavior of the Workspace.

Can you identify them in your Workspace? (*Expand below for answer*)

:::spoiler
![Workspace Components](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-06.png)
:::

## Workspace Commands
Commands are actions that can be executed in the containers of the Workspace pod.

The Workspace commands can be found in the `Workspace Panel` located on the right side of the IDE.

![Workspace Commands](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-07.png)

The [Devfile](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/devfile.yaml) used to create the Workspace of this workshop has multiple commands defined in it.

- Package the application
- Start Quarkus in dev mode
- Build native app
- Login to OpenShift
- Build container image
- Create OpenShift project and deploy

### Running the application

To run the application, open the `Workspace Panel` and click on the `Start Quarkus in dev mode` item.

A new panel will open at the bottom of your IDE and will execute the following command in the `maven` container of the Workspace pod.

```bash
mvn compile quarkus:dev -Dquarkus.http.host=0.0.0.0 -Dquarkus.live-reload.instrumentation=false -f ${CHE_PROJECTS_ROOT}/code-with-quarkus
```

Once the application is running, CodeReady Workspaces will detect that the process is listening on port 8080 and it will display a prompt asking the user if it should open the endpoint in a browser window or in the preview panel.

![Running the application](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-08.png)

Click on the button labeled `Open in Preview` and the preview panel will be displayed.

![Preview Panel](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-09.png)

:::info
CodeReady Workspaces does all the heavy lifting for you by creating OpenShift resources for your Workspace elements.

If the endpoint defined in the Devfile is configured as public, CodeReady Workspaces will automatically create an [OpenShift Route](https://docs.openshift.com/container-platform/4.8/rest_api/network_apis/route-route-openshift-io-v1.html) and make the service externally available.
:::

### Modify the application

Since we are using [Quarkus](https://quarkus.io/guides/getting-started#running-the-application) in this example, the command used to start the application is configured to run in development mode.

This enables hot deployment with background compilation, which means that when you modify your Java files and/or your resource files and refresh your browser, these changes will automatically take effect without having to restart the application.

Open the file named `GreetingResource.java` that is located in the path `src/main/java/org/acme` of the project.

Add a new method to the GreetingResource class below the `hello` method using the following code:

```java=
    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/greeting/{name}")
    public String greeting(@PathParam String name) {
        return "Hello, " + name;
    }
```

You will notice that the `@PathParam` annotation will have a red line below it. This means that the annotation has not been imported.

To quickly fix this issue we can use the CodeReady Workspaces built-in code assistance feature.

Hover over the `@PathParam` annotation and a pop-up dialog will be displayed. Click on the `Quick Fix...` option and the editor will suggest importing the `PathParam` annotation.

Select the option from the `org.jboss.resteasy.annotations` package.

![Quick Fix](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-10.png)

To verify that this change has been applied, open a new terminal from the Workspace Panel and execute the following command:

```bash=
curl http://localhost:8080/hello/greetings/John
```

The service should reply with the following text:

> Hello, John

### Attaching a debugger to the application

The [Devfile](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/devfile.yaml) used to create the Workspace of this workshop has [VS Code launch configuration](https://code.visualstudio.com/docs/editor/debugging) to attach a remote debugger to the application.

```yaml=
- name: Attach remote debugger
    actions:
      - referenceContent: |
          {
            "version": "0.2.0",
            "configurations": [
              {
                "type": "java",
                "request": "attach",
                "name": "Attach to Remote Quarkus App",
                "hostName": "localhost",
                "port": 5005
              }
            ]
          }
```

To attach the debugger to the application start the debug mode from the top menu '*Run > Start Debugging*'.

![Attach Debugger](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-11.png)

Add a breakpoint to the `GreetingResource.java` file by clicking to the left of the line number as shown below:

![Add breakpoint](https://raw.githubusercontent.com/jlmayorga/crw-workshop-quarkus/main/docs/images/crw-12.png)

Open a new terminal from the Workspace Panel and execute the following command:

```bash=
curl http://localhost:8080/hello/greetings/John
```

The service should reply with the following text:

> Hello, John






























