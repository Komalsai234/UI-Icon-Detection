# UI Icon Detection Application Documentation

## Overview
The UI Icon Detection application is a machine learning-based system for detecting UI elements (e.g., icons, text, buttons) in mobile phone screen images using a YOLOv10 model. The application comprises:

- **Backend:** A FastAPI server that processes images using a pre-trained YOLOv10 model, containerized with Docker and deployed on Azure Web App at https://test-hwi-gngcecf4ccccccbt.canadacentral-01.azurewebsites.net.

- **Frontend:** An HTML-based interface for uploading images and visualizing detection results, hosted on Azure Static Web App at https://uiicondetectionstorage.z13.web.core.windows.net/.

- **Dataset:** A structured dataset with Android and iPhone images, annotations, and a processed dataset for training and inference.

The system supports single-image uploads (PNG/JPG), processes them to detect UI elements, and displays bounding boxes with labels and confidence scores.

## Project Structure
The project is organized as follows:

```
ui_icon_detector/
├── Dataset/
│   ├── Android/
│   │   ├── Annotations/         # XML annotations for Android images
│   │   ├── JPEGImages/         # Android images
│   ├── iphone/
│   │   ├── Annotations/         # XML annotations for iPhone images
│   │   ├── JPEGImages/         # iPhone images
│   ├── Final_Processed_Dataset/
│   │   ├── test/               # Test images and labels
│   │   ├── train/              # Training images and labels
│   │   ├── val/                # Validation images and labels
│   │   ├── data.yaml           # YOLO dataset configuration
│   ├── Final_Training_Results/
│   │   ├── yolov10_icon_detection_v3/
│   │   │   ├── weights/
│   │   │   │   ├── best.pt     # Trained YOLOv10 model
├── frontend/
│   ├── index.html              # Frontend HTML with Tailwind CSS and JavaScript
├── src/
│   ├── config/
│   │   ├── config.py           # Configuration settings
│   ├── data/
│   │   ├── annotation_parser.py, data_splitter.py, dataset_loader.py, yolo_converter.py
│   ├── model/
│   │   ├── evaluator.py, exporter.py, inferencer.py, model_loader.py, trainer.py
│   ├── pipeline/
│   │   ├── dataset_preparation.py, export.py, inference.py, model_training.py, orchestrator.py, prediction/, visualization.py
│   ├── utils/
│   │   ├── logger.py, yaml_utils.py
│   ├── visualization/
│   │   ├── confusion_matrix.py
├── main.py                     # FastAPI application
├── requirements.txt            # Python dependencies
├── Dockerfile                  # Docker configuration
```

## Setup Instructions

### Backend Setup (FastAPI on Azure Web App via Azure Portal)

1. **Clone the Repository Locally**:
   - On your local machine, clone the repository:
     ```bash
     git clone <repository-url>
     cd ui_icon_detector
     ```

2. **Build the Docker Image**:
   - Ensure the `Dockerfile` and `requirements.txt` are in the root directory.
   - Build the Docker image:
     ```bash
     docker build -t ui-icon-detector .
     ```
   - Test locally (optional):
     ```bash
     docker run -p 8000:8000 ui-icon-detector
     ```
     Verify at `http://localhost:8000/docs`.

3. **Push to Azure Container Registry (ACR)**:
   - **Create ACR in Azure Portal**:
     - Log in to `https://portal.azure.com`.
     - Navigate to **Create a resource** → **Container Registry**.
     - Fill in:
       - Subscription, Resource Group, Registry Name (e.g., `<acr-name>`), Location (e.g., Canada Central), SKU (Basic).
     - Click **Review + Create** → **Create**.
     - Note the **Login server** (e.g., `<acr-name>.azurecr.io`).
   - **Enable Admin User**:
     - Go to **Container Registries** → `<acr-name>` → **Access keys**.
     - Enable **Admin user** and note the **Username** and **Password**.
   - **Push the Image**:
     - Log in to ACR locally:
       ```bash
       docker login <acr-name>.azurecr.io --username <acr-username> --password <acr-password>
       ```
     - Tag the image:
       ```bash
       docker tag ui-icon-detector <acr-name>.azurecr.io/ui-icon-detector:latest
       ```
     - Push:
       ```bash
       docker push <acr-name>.azurecr.io/ui-icon-detector:latest
       ```

4. **Deploy to Azure Web App**:
   - **Create Web App**:
     - In the Azure Portal, go to **App Services** → **Create Web App**.
     - Fill in:
       - Subscription, Resource Group.
       - Name: `test-hwi-gngcecf4ccccccbt`.
       - Publish: **Docker Container**.
       - Operating System: **Linux**.
       - Region: Canada Central.
       - App Service Plan: Choose or create a plan (e.g., B1 or higher).
     - In the **Docker** tab:
       - Options: **Single Container**.
       - Image Source: **Azure Container Registry**.
       - Registry: Select `<acr-name>`.
       - Image and Tag: `ui-icon-detector:latest`.
     - Click **Review + Create** → **Create**.
   - **Verify Deployment**:
     - Go to **App Services** → `test-hwi-gngcecf4ccccccbt` → **Overview**.
     - Click the **Default Domain** (`https://test-hwi-gngcecf4ccccccbt.canadacentral-01.azurewebsites.net`).
     - Visit `/docs` to confirm the FastAPI interface loads.


### Frontend Setup (Azure Blob Storage Static Website)

1. **Prepare the Frontend**:
   - Ensure `frontend/index.html` has the correct backend URL:
     ```javascript
     const backendUrl = 'https://test-hwi-gngcecf4ccccccbt.canadacentral-01.azurewebsites.net/api';
     ```

2. **Set Up Azure Blob Storage Static Website**:
   - **Create Storage Account (if not already created)**:
     - In the Azure Portal, go to **Create a resource** → **Storage Account**.
     - Fill in:
       - Subscription, Resource Group.
       - Storage Account Name: `uiicondetectionstorage`.
       - Location: Canada Central (or closest region).
       - Performance: Standard.
       - Redundancy: Geo-redundant storage (GRS) or as needed.
     - Click **Review + Create** → **Create**.
   - **Enable Static Website**:
     - Go to **Storage Accounts** → `uiicondetectionstorage` → **Data management** → **Static website**.
     - Toggle **Static website** to **Enabled**.
     - Set **Index document name** to `index.html`.
     - (Optional) Set **Error document path** (e.g., `404.html` if you have one).
     - Click **Save**.
     - Note the **Primary endpoint**: `https://uiicondetectionstorage.z13.web.core.windows.net/`.
     - A `$web` container is automatically created.
   - **Upload `index.html`**:
     - Go to **Storage Accounts** → `uiicondetectionstorage` → **Containers** → `$web`.
     - Click **Upload**.
     - Select `frontend/index.html` from your local machine.
     - Ensure it’s uploaded to the root of the `$web` container (not in a subdirectory).
     - Click **Upload**.
   - **Verify Access**:
     - Visit `https://uiicondetectionstorage.z13.web.core.windows.net/`.
     - Confirm the UI loads with a drag-and-drop area.

3. **Test the Frontend**:
   - Access `https://uiicondetectionstorage.z13.web.core.windows.net/`.
   - Ensure the interface displays correctly.


## Usage Instructions

### Using the Application

1. **Access the Frontend**:
   - Open `https://uiicondetectionstorage.z13.web.core.windows.net/` in a browser.
   - The interface shows a drag-and-drop area, a disabled "Process Image" button, and a dark mode toggle.

2. **Upload an Image**:
   - Drag and drop a PNG/JPG image (e.g., from `Dataset/test/images/`) or click "Browse".
   - The drop zone displays the file name, and the "Process Image" button enables.

3. **Process the Image**:
   - Click "Process Image".
   - The image is sent to `https://test-hwi-gngcecf4ccccccbt.canadacentral-01.azurewebsites.net/api/predict`.
   - A loader and progress bar appear.

4. **View Results**:
   - The image is displayed with bounding boxes, labels (e.g., "Icon", "Text"), and confidence scores.
   - Hover over the image to see the filename.
   - **Classes**: BackgroundImage, CheckedTextView, Icon, EditText, Image, Text, TextButton, Drawer, PageIndicator, UpperTaskBar, Modal, Switch.