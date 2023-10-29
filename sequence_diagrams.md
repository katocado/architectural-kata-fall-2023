# Requirements

### Functional Requirements

1. The camera footage should be stored within SD card before uploading to the frontend application (mobile or desktop) or application server. 
2. The camera should be able to send notification to the frontend application (mobile or desktop) via LoraWan, 3G, or Satellite. 
3. The camera should be able to report its health status to frontend application (mobile or desktop). 
4. The camera should be able to report species identification to frontend application (mobile or desktop).
5. The camera should be able to store edge models. 
6. The camera should be able to use the edge models to perform species analysis and identification. 
7. The camera should record a 5 to 10 seconds video when a motion is detected. 
8. The user should be able to retrieve video footage from camera via SD card, LoraWan, 3G, or Satellite. 
9. The user should be able to store video footage offline in the mobile or desktop.
10. The user should be able to upload video footage to application server (via frontend application - mobile or desktop).
11. The user should be able to discover nearby camera (via frontend application - mobile or desktop).
12. The user should be able to connect to a specific camera (via frontend application - mobile or desktop).
13. The user should be able to receive notifications from the camera (via frontend application - mobile or desktop).
14. The user should be able to view the camera status details (via frontend application - mobile or desktop).
15. The user should be able to control the camera (via frontend application - mobile or desktop).
16. The user should be able to update the camera settings or edge model (via frontend application - mobile or desktop).
17. The user should be able to perform further analysis (via frontend application - mobile or desktop).
18. The user should be able to send the video footage to iNaturalist for help (via frontend application - mobile or desktop).
19. The user should be able to publish the species occurrences to GBIF using the CamTrap DB data exchange format (via frontend application - mobile or desktop).
20. Geoprivacy: https://www.inaturalist.org/pages/help#geoprivacy

## Camera Focus

### Main Components: 
1. **MotionDetector** - A component that is responsible to detect motion and notify other component 
2. **CameraController** - A controller that is responsible to to coordinate the request and response between different components
3. **CameraRecordingService** - A service that is responsible to perform recoding
4. **CameraEdgeModelService** - A service that is responsible for using AI edge model to perform footage species labelling
5. **CameraLocalStorage** - A data access layer that is responsible for storing or accessing data
6. **CameraVideoService** - A service that is responsible for reading and writing video
7. **CameraHealthService** - A service that is responsible for reading the camera health statuses

### Sequence Digrams:

#### Captures and labels footage (1, 2, 4, 5, 6, 7)

```mermaid
sequenceDiagram

    actor Object
    participant MotionDetector
    participant CameraController
    participant CameraRecordingService
    participant CameraEdgeModelService
    participant CameraLocalStorage
    participant ConnectedFrontendApp

    Object->>MotionDetector: Move within the range

    MotionDetector->>CameraController: Send recording request

    CameraController->>CameraRecordingService: Send recording request

    CameraRecordingService->>CameraRecordingService: Perform recording

    CameraRecordingService->>CameraLocalStorage: Store the recorded footage

    CameraRecordingService-->>CameraController: Return footage information

    CameraController->>CameraEdgeModelService: Send labelling request

    CameraEdgeModelService->>CameraLocalStorage: Retrieve footage 
    CameraLocalStorage-->>CameraEdgeModelService: Return footage 

    CameraEdgeModelService->>CameraEdgeModelService: Perform labelling

    CameraEdgeModelService->>CameraLocalStorage: Update the footage information with label

    CameraEdgeModelService-->>CameraController: Return footage labelling result

    alt is connected to Frontend application
        CameraController->>ConnectedFrontendApp: Publish events
    else is not connected to Frontend application
        CameraController->>CameraStorage: Store events
        Note over CameraController,CameraStorage: The device can poll for stored events once the connection is established
    end

```

#### Retrieves footage from camera device (8)

```mermaid
sequenceDiagram

    actor User
    participant ConnectedFrontendApp
    participant CameraController
    participant CameraVideoService
    participant CameraLocalStorage

    User->>ConnectedFrontendApp: Initiated read footages request

    ConnectedFrontendApp->>CameraController: Send read footages request

    CameraController->>CameraVideoService: Send read footages request

    CameraVideoService->>CameraLocalStorage: Retrieve footages metadata or file (based on the request type)
    CameraLocalStorage-->>CameraVideoService: Return results

    CameraVideoService-->>CameraController: Return results

    CameraController-->>ConnectedFrontendApp: Return results

    ConnectedFrontendApp-->>User: Display results

```

#### Reports camera device health status (3)

```mermaid
sequenceDiagram

    actor User
    participant ConnectedFrontendApp
    participant CameraController
    participant CameraHealthService

    CameraHealthService->>CameraHealthService: Read camera device health statues
    Note right of CameraHealthService: Listening on camera device events or read camera device information periodically

    CameraHealthService-->>CameraController: Send camera device health statues

    CameraController-->>ConnectedFrontendApp: Send camera device health statues

    ConnectedFrontendApp->>ConnectedFrontendApp: Analyse camera device health statues

    alt Good health status
        ConnectedFrontendApp-->>User: Update camera device information
    else Bad health status
        ConnectedFrontendApp-->>User: Update camera device information and trigger notification
    end
```

## Frontend Application (Mobile, Web, Desktop) Focus

### Main Components: 

1. **FEAppUI** - A user interface that is responsible to provide a way for user to use the app
2. **FEAppController** - A controller that is responsible to coordinate the request and response between different components
3. **FEAppCameraDiscoveryService** - A service that is responsible to discover camera device
4. **FEAppCameraConnectorService** - A service that is responsible to establish a connection between the mobile/desktop device with camera device
5. **FEAppCameraEventListener** - An event listener that is responsible to accept events from connected camera device
6. **FEAppCameraControlService** - A service that is responsible to control the camera device, including update the edge model, turn on/off the camera device, update the camera device settings, etc.
7. **FEAppVideoUploadService** - A service that is responsible to upload local device footage to the backend server
8. **FEAppLocalStorage** - A data access layer that is responsible for storing or accessing data

### Sequence Diagrams

#### Connect to camera (11, 12)

```mermaid
sequenceDiagram

    actor User
    participant FEAppUI
    participant FEAppController
    participant FEAppCameraDiscoveryService
    participant FEAppCameraConnectorService
    participant Camera

    User->>FEAppUI: Initial camera searching process
    FEAppUI-->>User: Display loader

    FEAppUI->>FEAppController: Send camera searching request

    FEAppController->>FEAppCameraDiscoveryService: Send camera searching request

    Camera-->>FEAppCameraDiscoveryService: Signal

    FEAppCameraDiscoveryService->>FEAppCameraDiscoveryService: Process camera signal

    FEAppCameraDiscoveryService-->>FEAppController: Return camera device information

    FEAppController-->>FEAppUI: Return camera device information

    FEAppUI-->>User: Display camera device information
    User->>FEAppUI: Select camera device to connect
    FEAppUI-->>User: Display loader 

    FEAppUI->>FEAppController: Send connect camera request

    FEAppController->>FEAppCameraConnectorService: Send connect camera request

    FEAppCameraConnectorService-->Camera: Establish a connection

    FEAppCameraConnectorService-->>FEAppController: Return connection status

    FEAppController-->>FEAppUI: Return connection status

    FEAppUI-->>User: Notify or display connection status

```

#### Receives camera events (13, 14)

```mermaid
sequenceDiagram

    actor User
    participant FEAppUI
    participant FEAppController
    participant FEAppCameraEventListener
    participant Camera

    Note over User,Camera: Establish connection (Please refer to sequence diagram 1)
    create participant ConnectedCamera
    User-->ConnectedCamera: Connection Established

    ConnectedCamera-->>FEAppCameraEventListener: Publish camera event
    Note over ConnectedCamera,FEAppCameraEventListener: Events can be health status, species identification result, settings changes, etc 

    FEAppController->>FEAppCameraEventListener: Send start listening request

    FEAppCameraEventListener->>FEAppCameraEventListener: Listen and process events

    FEAppCameraEventListener-->>FEAppController: Return camera event information

    FEAppController-->>FEAppUI: Return camera event information

    FEAppUI-->>User: Notify or display event information

```

#### Control or update the connected camera (15, 16)

```mermaid
sequenceDiagram

    actor User
    participant FEAppUI
    participant FEAppController
    participant FEAppCameraControlService
    participant Camera

    Note over User,Camera: Establish connection (Please refer to sequence diagram 1)
    create participant ConnectedCamera
    User-->ConnectedCamera: Connection Established

    User->>FEAppUI: Initial camera action
    FEAppUI-->>User: Display loader

    FEAppUI->>FEAppController: Send camera action request

    FEAppController->>FEAppCameraControlService: Send camera action request

    FEAppCameraControlService->>FEAppCameraControlService: Process and validate camera action request
    
    FEAppCameraControlService->>ConnectedCamera: Publish processed camera action
    Note over User,ConnectedCamera: Listening on the corresponding action status event, then notify or display the action status (Please refer to sequence diagram 2)

```

#### Upload footage to backend server (9, 10)

```mermaid
sequenceDiagram

    actor User
    participant FEAppUI
    participant FEAppController
    participant FEVideoUploadService
    participant FELocalStorage
    participant BEApp

    User->>FEAppUI: Initial upload footage process

    FEAppUI->>FEAppController: Send upload footage request

    FEAppController->>FELocalStorage: Retrieve footage metadata
    FELocalStorage-->>FEAppController: Return footage metadata

    FEAppController->>FEAppController: Process footage metadata

    FEAppController->>FEVideoUploadService: Send upload request along with footage metadata

    loop Upload footage chunk by chunk
        par 
            FEVideoUploadService->>FELocalStorage: Retrieve footage file chunk
            FELocalStorage->>FEVideoUploadService: Return footage file chunk
        and
            FEVideoUploadService->>FEVideoUploadService: Process footage file chunk
        and
            FEVideoUploadService->>BEApp: Upload footage file chunk
            BEApp-->>FEVideoUploadService: Return upload status
        end
    end

    FEVideoUploadService-->>FEAppController: Return upload status summary

    FEAppController-->>FEAppUI: Return upload status summary

    FEAppUI-->>User: Notify or display upload status summary
    
```

## Backend Application Focus

### Main Components:
1. **BEAppService** - A general backend service (might contain sub services) that responsible to handle the request or response between different apps or services
2. **BETrapLabellingService** - A service that is responsible to perform camera trap labelling
3. **BEFootageDatabase** - A data access layer that is responsible for storing or accessing data
4. **BEFootageBlobStorage** - A data access layer that is responsible for storing or accessing data
5. **BEDatabase** - A general data access layer (might contain sub services) that is responsible for storing or accessing data

### Sequence Diagrams

#### Label and analyse footage (17, 18, 19)

```mermaid
sequenceDiagram

    actor User
    participant FEApp
    participant BEAppService
    participant BETrapLabellingService
    participant BEFootageDatabase
    participant BEFootageBlobStorage

    User->>FEApp: Initial labelling process

    FEApp->>BEAppService: Send labelling request

    BEAppService->>BEFootageDatabase: Retrieve footage metadata
    BEFootageDatabase-->>BEAppService: Return footage metadata

    BEAppService->>BEFootageBlobStorage: Retrieve footage file
    BEFootageBlobStorage-->>BEAppService: Return footage file

    BEAppService->>BETrapLabellingService: Send footage file for trap labelling
    BETrapLabellingService-->>BEAppService: Return labelling result

    BEAppService->>BEFootageDatabase: Update footage metadata

    BEAppService-->>FEApp: Return labelling process summary

    FEApp-->>User: Notify or display labelling process summary

```

#### Geoprivacy data access (20)

```mermaid
sequenceDiagram

    actor User
    participant FEApp
    participant BEAppService
    participant BEDatabase

    User->>FEApp: View information (any kind of information)

    FEApp->>BEAppService: Send retrieve information request

    BEAppService->>BEAppService: Validate user access token and location 

    BEAppService->>BEAppService: Validate user agent 

    alt The user has permission
        BEAppService->>BEDatabase: Retrieve data
        BEDatabase-->>BEAppService: Return data

        BEAppService-->>FEApp: Return data 

        FEApp-->>User: Notify or display data
    else The user has no permission
        BEAppService-->>FEApp: Return access denied

        FEApp-->>User: Display access denied
    end

```