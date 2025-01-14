# Azure Video Analyzer sample to capture and record live video from an RTSP camera behind a firewall  

Azure Video Analyzer sample to capture and record live video from an RTSP camera behind a firewall. 

### Contents

| File             | Description                                                   |
|-------------------------|---------------------------------------------------------------|
| `PrivateCameraPipelineSampleCode.csproj`| Project file                                                 |
| `Program.cs`            | The main program file                                         |

### Suggested Pre-reading
* [Connect camera to cloud](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/cloud/connect-cameras-to-cloud)
* [Connect camera to cloud using remote device adapter](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/cloud/use-remote-device-adapter)

### Pre-requisites

1. An Azure account that includes an active subscription. [Create an account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) for free if you don't already have one.
    * Get your Azure Active Directory [Tenant Id](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-how-to-find-tenant)
    * Register an application with Microsoft identity platform to get app registration [Client Id](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app#register-an-application) and [Client secret](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app#add-a-client-secret)

1. Create a [Video Analyzer account](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/create-video-analyzer-account?tabs=portal), and attach an IoT Hub to this account.

1.  [IoT Edge device with the Video Analyzer edge module installed and configured](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/edge/deploy-iot-edge-device), under the IoT Hub used above.

1. [Visual Studio Code](https://code.visualstudio.com/) on your development machine with following extensions -
    * [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

1. [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1) on your development machine.

1. An RTSP [capable camera](https://aka.ms/service-supported-cameras) and the RTSP server on this camera should be behind a firewall (not accessible over the internet).
    * Ensure that camera(s) are on the same network as the above IoT Edge device
    * Ensure that you can configure the camera to send video at or below a maximum bandwidth of 3000 Kbps

1. [Create an IoT device](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/cloud/use-remote-device-adapter#create-an-iot-device)

1. [Create a remote device adapter](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/cloud/use-remote-device-adapter#create-a-remote-device-adapter)

### Setup

- Open your local clone of this git repository in Visual Studio Code.
- Go to `src\cloud-video-processing\capture-from-rtsp-camera-behind-firewall\Program.cs` and provide values for the following variables:

| Variable       | Description                                |
|----------------------|--------------------------------------------|
| SubscriptionId | Provide Azure subscription Id    |
| ResourceGroup | Provide resource group name |
| AccountName | Provide Video Analyzer account name |
| TenantId | Provide tenant id |
| ClientId | Provide app registration client id |
| Secret | Provide app registration client secret |
| AuthenticationEndpoint | Provide authentication end point (example: https://login.microsoftonline.com) |
| ArmEndPoint | Provide ARM end point (example: https://management.azure.com) |
| TokenAudience | Provide token audience (example: https://management.core.windows.net) |
| PrivateCameraTunnelingDeviceId | Provide the Device Id for your camera used for registration |
| IotHubNameForPrivateCamera | Provide IoT Hub name |
| PrivateCameraSourceRTSPURL | Provide RTSP source URL specified as **localhost** (example:`rtsp://localhost:554/<camera-specific-suffix>`)  |
| PrivateCameraSourceRTSPUserName | Provide RTSP source username |
| PrivateCameraSourceRTSPPassword | Provide RTSP source password |
| PrivateCameraVideoName | Provide unique video name to capture live video from this RTSP source |

- Optionally, you can provide custom values for topology and pipeline parameters, defined just below the variables mentioned in the table (lines 39 - 46).
- Save the changes.

### Code walkthrough

In this section, we will be describing the steps in Program.cs

The Main() is the starting point with two function calls:

```
public static async Task Main(string[] args)
{
    await SetupClientAsync();
    await IngestFromPrivateCameraAsync();
}
```

- SetupClientAsync() method is used for service principal authentication.
- IngestFromPrivateCameraAsync() method is used to capture (ingest) and record video from an RTSP camera behind a firewall. This method does the following:

    1. Create a [pipeline topology](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/pipeline) in `CreateTopologyForPrivateCameraAsync()` method with the following nodes:
        *  RTSP source node
        *  Video sink node

    1. n successful creation of that topology, a [live pipeline](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/pipeline) is created in the `CreateLivePipelineForPrivateCameraAsync()` method using: 
        * Topology and Pipeline names defined in the variables section during setup.
        *  `bitrateKbps` - Bitrate is set to 1500 kbps by default in line 282. This represents the maximum bitrate for the RTSP camera, and it must be between 500 and 3000 Kbps. If bitrate of the live video from the camera exceeds this threshold, then the service will keep disconnecting from the camera, and retrying later - with exponential backoff.

    1. If the pipeline is created successfully, it is then activated using ActivateLivePipelineAsync() method. This will start the flow of live video.

### Running the sample

Once you have the configuration steps completed, you can run the program.

- Start a debugging session. 
    * You can set this project as default project to run on hitting F5 by modifying the files in .vscode folder. Learn more about [debugging in VS Code](https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md).
        * `launch.json` - Update the **"program"** and **"cwd"** to launch PrivateCameraPipelineSampleCode.
        * `tasks.json` - Update **"args"** to point to PrivateCameraPipelineSampleCode.csproj.
    * Alternatively, go to TERMINAL window in the Visual Studio Code, navigate using `cd <path>` to src/cloud-video-processing/capture-from-rtsp-camera-behind-firewall folder. Type commands `dotnet build` and `dotnet run` to compile and run the program respectively.
- You will start seeing some messages printed in the TERMINAL window regarding topology and pipeline creation. If the creation is successful, the live pipeline is activated and you can go to the Azure portal to view the video.
- Login to [Azure portal](https://portal.azure.com/), go to the Video Analyzer account being used for this project.
- Click on Videos blade and choose the video created. The video will be in an `In use` status.  Click on the video, and you should see a [low latency stream](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/playback-recordings-how-to#low-latency-streaming) of the live video from the camera.
- Go back to Visual Studio Code TERMINAL window and press enter to deactivate the pipeline and cleanup the resources including pipeline and topology. The recording is persisted and status changes to `Not recording`.

### Next steps

- [Export a portion of the recorded video as an MP4 file](./src/video-export)
- Learn more about [pipelines](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-docs/pipeline)
- [Quotas and limitations](https://aka.ms/livequota) on live pipelines
