# Remote Debugging of Containers

## Task 1: Expose Debug Ports

To begin with, we will expose the debug port used by Node.js. The default debug port is `9229`.

### Step 1.1: Identify the Debug Port

Different programming languages use different ports for debugging:

- **Node.js**: `9229` (when using the `--inspect` flag)
- **Python (debugpy)**: `5678`
- **Java (JDWP)**: `5005`

Since we are focusing on Node.js, we will use port `9229`.

### Step 1.2: Run the Container with Debug Port

To run the container and expose both the application and debug ports, use the command below:

```bash
podman run -d -p 3000:3000 -p 9229:9229 --name debug-container my-node-app
````

* `-p 3000:3000` maps the app's port
* `-p 9229:9229` maps the debug port

**Note**: If port `9229` is already in use, you can modify the host port like so: `-p 9230:9229`

## Task 2: Mount Source Code for Live Debugging

Now we will mount the local source code directory to the container so any code changes are reflected live.

### Step 2.1: Volume Mount the Local Directory

Use the `-v` flag to mount your current directory to the `/app` path inside the container:

```bash
podman run -d -p 3000:3000 -p 9229:9229 -v $(pwd):/app --name debug-container my-node-app
```

### Step 2.2: Verify if Files Are Mounted

To confirm the files are accessible inside the container, run:

```bash
podman exec -it debug-container ls /app
```

You should see the contents of your current working directory.

## Task 3: Connect IDE Debugger to Container

In this step, we'll configure VS Code to attach to the running container and use breakpoints.

### Step 3.1: Install Required VS Code Extensions

You need the following extensions installed in VS Code:

* **Remote - Containers**
* **Debugger for Node.js**

### Step 3.2: Create a Debug Configuration

Open your project folder in VS Code and go to the **Run and Debug** panel. Then:

1. Click **Create a launch.json file**
2. Select **Node.js** as the environment
3. Replace the configuration with:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Container",
      "address": "localhost",
      "port": 9229,
      "localRoot": "${workspaceFolder}",
      "remoteRoot": "/app"
    }
  ]
}
```

Click the green play button to start the debugger.

## Troubleshooting

* If the container doesn’t start, make sure the ports are not already in use.
* If your source code doesn't show up inside the container, verify the path in the `-v` flag is correct.
* If VS Code doesn’t attach:

  * Confirm the container is running.
  * Ensure port `9229` is open.
  * Check if the Node.js app is running with the `--inspect` or `--inspect-brk` flag.

