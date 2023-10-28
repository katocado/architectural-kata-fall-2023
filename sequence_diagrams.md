# Camera Features

### Main Components: 
1. **MotionDetector** - A component that is responsible to detect motion and notify other component 
2. **CameraController** - A controller that is responsible provide an interface to interact with other components
3. **CameraRecordingService** - A service that is responsible to perform recoding
4. **CameraEdgeModelService** - A service that is responsible for using AI edge model to perform footage species labelling
5. **CameraLocalStorage** - A data access layer that is responsible for storing or accessing data
6. **CameraVideoService** - A service that is responsible for reading and writing video
7. **CameraHealthService** - A service that is responsible for reading the camera health statuses

### Sequence Digrams (Features):

#### Captures and labels footage

```mermaid
sequenceDiagram

    actor Object
    participant MotionDetector
    participant CameraController
    participant CameraRecordingService
    participant CameraEdgeModelService
    participant CameraLocalStorage
    participant ConnectedFrontendApplication

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
        CameraController->>ConnectedFrontendApplication: Publish events
    else is not connected to Frontend application
        CameraController->>CameraStorage: Store events
        Note over CameraController,CameraStorage: The device can poll for stored events once the connection is established
    end

```

#### Read footages

```mermaid
sequenceDiagram

    actor User
    participant ConnectedFrontendApplication
    participant CameraController
    participant CameraVideoService
    participant CameraLocalStorage

    User->>ConnectedFrontendApplication: Initiated read footages request

    ConnectedFrontendApplication->>CameraController: Send read footages request

    CameraController->>CameraVideoService: Send read footages request

    CameraVideoService->>CameraLocalStorage: Retrieve footages metadata or file (based on the request type)
    CameraLocalStorage-->>CameraVideoService: Return results

    CameraVideoService-->>CameraController: Return results

    CameraController-->>ConnectedFrontendApplication: Return results

    ConnectedFrontendApplication-->>User: Display results

```

#### Health status reporting

```mermaid
sequenceDiagram

    actor User
    participant ConnectedFrontendApplication
    participant CameraController
    participant CameraHealthService

    CameraHealthService->>CameraHealthService: Read camera device health statues
    Note right of CameraHealthService: Listening on camera device events or read camera device information periodically

    CameraHealthService-->>CameraController: Send camera device health statues

    CameraController-->>ConnectedFrontendApplication: Send camera device health statues

    ConnectedFrontendApplication->>ConnectedFrontendApplication: Analyse camera device health statues

    alt Good health status
        ConnectedFrontendApplication-->>User: Update camera device information
    else Bad health status
        ConnectedFrontendApplication-->>User: Update camera device information and trigger notification
    end
```

# Frontend Application (Mobile, Web, Desktop)

### Establish connection

```mermaid
sequenceDiagram

    actor User
    participant FEView As Frontend Application View (UI)
    participant FECameraDiscoveryService As Frontend Application Camera Discovery Service
    participant FECameraConnector As Frontend Application Camera Connector
    participant Camera

    User->>FEView: Search camera(s)
    FEView--)User: Display loader

    FEView-)FECameraDiscoveryService: Initial searching process

    Camera--)FECameraDiscoveryService: Signal(s)

    FECameraDiscoveryService-)FECameraDiscoveryService: Process signal(s) and metadata

    FECameraDiscoveryService--)FEView: Camera(s) details

    FEView--)User: Display camera(s) details
    User->>FEView: Select camera(s) to connect
    FEView--)User: Display loader 

    FEView-)FECameraConnector: Connect to selected camera(s)

    FECameraConnector-->Camera: Establish a connection

    FECameraConnector--)FEView: Connection established

    FEView--)User: Notify and display connection established

```

### Listening on connected camera event(s)

```mermaid
sequenceDiagram

    actor User
    participant FEView As Frontend Application View (UI)
    participant FECameraEventListener As Frontend Application Camera Event Listener
    participant Camera

    Note over User,Camera: Establish connection (Please refer to sequence diagram 1)
    create participant ConnectedCamera As Connected Camera
    User-->ConnectedCamera: Connection Established

    ConnectedCamera--)FECameraEventListener: Raw events
    Note over ConnectedCamera,FECameraEventListener: Events can be health status, species identification, camera settings changes, etc 

    FECameraEventListener-)FECameraEventListener: Process events

    FECameraEventListener--)FEView: Processed events

    FEView--)User: Notify or display updated data

```

### Control connected camera

```mermaid
sequenceDiagram

    actor User
    participant FEView As Frontend Application View (UI)
    participant FECameraController As Frontend Application Camera Controller
    participant Camera

    Note over User,Camera: Establish connection (Please refer to sequence diagram 1)
    create participant ConnectedCamera As Connected Camera
    User-->ConnectedCamera: Connection Established

    User->>FEView: Perform action (update/control camera)
    FEView--)User: Display loader

    FEView->>FECameraController: Action details

    FECameraController->>FECameraController: Validate action
    
    FECameraController->>ConnectedCamera: Publish action
    Note over FECameraController,ConnectedCamera: Listening on the corresponding action status event (Please refer to sequence diagram 2)

    FECameraController--)FEView: Action status
    
    FEView--)User: Notify and display action status

```

### Upload local footages to backend server

```mermaid
sequenceDiagram

    actor User
    participant FEView As Frontend View (UI)
    participant FEStorage As Frontend Storage
    participant FEVideoUploader As Frontend Video Uploader
    participant BEServer As Backend Server

    User->>FEView: View local footage

    FEView->>FEStorage: Retrieve local footage details

    FEView--)User: Display local footage details
    User->>FEView: Select local footage to upload
    FEView--)User: Display loader

    FEView->>FEVideoUploader: Selected local footage details

    loop Upload selected footages
        par 
            FEVideoUploader->>FEStorage: Retrieve local footage video
            FEStorage->>FEVideoUploader: Local footage video bytes
        and
            FEStorage->>FEVideoUploader: Process and chunk local footage video
        and
            FEVideoUploader->>BEServer: Upload chunk
            BEServer--)FEVideoUploader: Upload status
        end
    end

    FEVideoUploader--)FEView: Upload status

    FEView--)User: Notify and display upload status
```

# Backend Application 
