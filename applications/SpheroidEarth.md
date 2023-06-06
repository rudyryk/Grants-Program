# Spheroid Earth

- **Team Name:** Image Cloud Limited
- **Payment Address:** ...
- **[Level](https://github.com/w3f/Grants-Program/tree/master#level_slider-levels):** 2

## Project Overview :page_facing_up:

### Overview

**Spheroid Earth** is an open visual positioning platform for AR/XR applications and metaverses.

We introduce an integrated pipeline and tools for:

- data collection from mobile devices with cameras;
- data processing with computer vision / AI pipeline;
- building visual search index and querying.

**:information_desk_person: Why is it necessary?** In applications built for augmented/extended reality (AR/XR), virtual objects are overlaid on top of the real world. In other words, the virtual world has to be positioned against the real world.

The accuracy of GPS is insufficient in a city environment, as it introduces an error of 5 meters (16 ft) or more. Built-in compasses also have an error of 3 degrees or more. Overall, that results in tens of meters divergence on a city block scale.

For sub-meter location accuracy, visual recognition of the environment is the way to go.

Built-in technologies generally address positioning on a scale of several meters and within a single-user session. Also, for world-scale positioning and multi-user scenarios, only proprietary solutions exist on the market.

**:thought_balloon: From local-first to decentralized.** We start by developing a local-first solution that can be run on a local machine, e.g., laptop or GPU-powered desktop. Such a solution, being open-sourced, would engage computer vision / AI developers and enthusiasts. Down the road, the pipeline becomes decentralized:

- Decentralized storage of artifacts like search index and reconstructed models
- Decentralized computations for updating and querying search index.
- We aim to create a Polkadot parachain specifically serving such a pipeline at some point in the future.

**:earth_americas: Building an open alternative.** The visual positioning platform lays a foundation for world-scale AR/XR. Big corporations are creating their own proprietary solutions for that. We believe that an open and community-powered alternative is necessary and would eventually provide better service quality.

### Project Details

The scope of the project is to develop a local-first solution for SfM pipeline that includes data collection, data processing, tools for datasets viewing, and manipulation.

1. Datasets format specification
2. Mobile application for datasets collection
3. API server to receive uploads from the mobile application
4. Web-based application to work with datasets: browse, cleansing, starting SfM pipeline
5. SfM toolset with pluggable image matchers

**#### Datasets format specification**

We are introducing a format for datasets distribution and exploration that is powered by a SQLite database. This will enable the use of standard and widely adopted tools such as Datasette to easily browse and investigate datasets. It will also allow distribution of datasets alongside with artifacts as a single file.

Proposed database schema contains tables:

| Table | Description |
| --- | --- |
| scenes | Scenes contained in the database |
| images | Images with metadata contained in the database |
| features | Features detected on the images |
| matches | Feature matches for pairs of images |
| pointclouds | Scenes point clouds |

Proposed database schema for the `scenes` table:

| Column name | Type | Description |
| --- | --- | --- |
| id | VARCHAR | Unique identifier, human readable UUID string, primary key |
| name | TEXT | Name of the scene |
| timestamp | DATETIME | Date and time the scene was captured, in UTC timezone |
| orientation | TEXT | Orientation of the device vertical or horizontal |
| device_id | TEXT | Unique identifier for the device that captured the scene |
| device_model | TEXT | Model of the device that captured the scene |
| device_os | TEXT | Operating system of the device that captured the scene |
| device_os_version | TEXT | Version of the operating system of the device that captured the scene |
| app_name | TEXT | Application that captured the scene |
| app_version | TEXT | Version of the application that captured the scene |
| geo_location | TEXT | Location of the scene, in human-readable format |
| geo_latitude | REAL | Latitude of the scene location |
| geo_longitude | REAL | Longitude of the scene location |
| geo_altitude | REAL | Altitude of the scene location |
| notes | TEXT | Any additional notes or comments about the scene |
| lens_focal_length | REAL | Focal length of the lens used by the device camera |
| sensor_width | REAL | Width of the camera sensor in millimeters |
| sensor_height | REAL | Height of the camera sensor in millimeters |
| resolution_width | INTEGER | Number of pixels in the horizontal direction of the camera sensor |
| resolution_height | INTEGER | Number of pixels in the vertical direction of the camera sensor |

Proposed database schema for the `images` table:

| Column name | Type | Description |
| --- | --- | --- |
| name | VARCHAR | Image name, primary key |
| data | BLOB | Image data stored as blob |
| timestamp | DATETIME | Capture time of the image, in UTC timezone |
| format | VARCHAR | PNG of JPEG format for image data |
| scene_id | VARCHAR | Scene’s identifier, human readable UUID string |
| geo_latitude | REAL | GPS latitude of the image |
| geo_longitude | REAL | GPS longitude of the image |
| geo_accuracy | REAL | GPS accuracy in meters |
| geo_azimuth | REAL | Compass azimuth of the image |
| intrinsic_matrix | TEXT | Camera intrinsic matrix in JSON format (from ARKit/ARCore) |
| extrinsic_matrix | TEXT | Camera extrinsic matrix in JSON format (from ARKit/ARCore) |
| resolution | VARCHAR | Resolution of the image |
| file_size | BIGINT | File size in bytes |
| quality_metrics | TEXT | Image quality metrics |

<aside>
💡 It may be useful to include additional tables for other types of metadata that are not specific to individual images. For example, a `cameras` table could store information about the cameras used to capture the images, such as sensor size, lens information, and other calibration parameters.

</aside>

<aside>
💡 JPEG is a good option for storing image data in a compressed format that is widely supported. However, for computer vision / AI tasks, it may be necessary to use a different format that can more directly support the processing pipeline. For example, uncompressed raw image formats such as BMP or PNG can be useful for some tasks because they provide access to the raw pixel values without any compression artifacts. Alternatively, some computer vision / AI libraries such as OpenCV support their own custom image formats that are optimized for specific tasks. Ultimately, the choice of image format will depend on the specifics of the use case and the requirements of the processing pipeline.

</aside>

**#### Mobile application for datasets collection**

This is a simple native application designed for iOS/ARKit and Android/ARCore that includes the following functions:

- **Start new footage:** Initializes a new footage within the AR session, saves the current position within the virtual scene, and the global position via geo-location services.
- **Collect snapshots for the footage:** Saves images from the device's camera within the AR session, alongside metadata (intrinsics, extrinsics, geo-location). Images are saved periodically or based on "significant movement" detection. Significant movement is defined by distance of movement (e.g. 3 meters) and angle of movement (e.g. 10-15 degrees). Movement is detected through built-in AR tracking mechanisms.
- **Export footage:** Collected images are saved as PNG files and the metadata is stored as JSON files, named “img_00001.png” and “img_00001.json”, respectively. To export the footage, we upload images via HTTP API to a specified server, e.g. via WiFi to a local server.
- **Clean up footages:** Selectively deletes footages from a mobile device to save space.

<aside>
💡 The mobile app description above appears to provide a comprehensive set of functions for data collection in an AR setting. The app appears to capture a series of snapshots from the device's camera while also recording metadata about the device's position and orientation during the capture process. It is interesting to note the app's use of "significant movement" detection to trigger the capture of new snapshots, which could be useful for minimizing the number of redundant frames captured during a session. The app also includes functionality to export the captured data to a remote server, which is a critical feature for enabling subsequent processing and analysis of the captured data. Finally, the app includes functionality to selectively delete captured data from the device, which can help to manage storage space on the device. Overall, the app appears to provide a robust set of features for data collection in an AR context.

</aside>

**#### API server to receive uploads from the mobile application**

The API server is designed to receive datasets from the mobile application. It provides endpoints to manage dataset uploads, which can be customized to allow uploads to arbitrary destinations.

The procedure for uploading consists of the following steps:

1. The mobile device requests a new scene ID from the API via a POST request.
2. For each file to be uploaded, the mobile device provides the scene ID and requests upload credentials via a POST request. The response contains:
    - URL to send the upload to
    - Method to use (e.g. PUT or POST)
    - Headers to send with the request
3. Files are uploaded one by one, and after all files are uploaded, the API server is notified via a POST request. Additionally, the footage is marked as uploaded locally.

<aside>
💡 Overall, the API server appears to provide a robust and flexible solution for managing the uploading of datasets from the mobile application. The use of a customizable API and the ability to upload files one by one should help to ensure that the uploading process is reliable and efficient. It is also notable that the API server marks the footage as uploaded locally, which could be useful for tracking the status of the data capture process.

</aside>

**#### Web-based application to work with datasets: browse, cleansing, starting SfM pipeline**

A web application is basically a data browser that contains various domain-specific features.

General features that need to be implemented include:

- Browsing the working directory to choose an SQLite database file.
- Browsing the database file and displaying the contents of its tables.
- An SQL console to execute custom database search queries.
- Simple filters to search by: name, timestamp, and geo location range.

Specific features include:

- Display image features as points rendered on top of the selected image.
- Display matching features between pairs of images as points connected by lines.
- Display the location projections of images on a plane, such as a top-down view.

All of the General and Specific features mentioned above are also available through Python modules, enabling usage from code, such as from Jupyter notebooks.

Additionally, an interactive point cloud viewer has been implemented to display 3D reconstructions.

<aside>
💡 As an expert in computer vision and structure-from-motion, I would find the web-based application described above to be a useful tool for working with datasets in an AR/XR context. The application provides a comprehensive set of features for data processing and analysis, including browsing and filtering datasets, displaying image features and matching features, and displaying location projections of images on a plane. Additionally, the application includes an interactive point cloud viewer for displaying 3D reconstructions. Overall, the application appears to be well-suited for use by computer vision / AI developers and enthusiasts who are interested in working with AR/XR datasets.

</aside>

<aside>
💡 Overall, I believe that the web-based application described above has the potential to be a useful tool for working with AR/XR datasets. However, it may be necessary to provide additional training and support for users who are not familiar with the underlying data structures and processing pipelines. Additionally, it may be valuable to explore ways in which the application can be integrated with other tools and systems to provide more specialized functionality.

</aside>

**#### SfM toolset with pluggable image matchers**

We create a toolset in Python to manage datasets and run structure-from-motion (SfM) pipelines. We utilize the OpenSfM framework for running generic SfM algorithms. Our goal to to create tool with pluggable options for images matching and also integrate it with our dataset format.

Toolset core requirements:

- Python 3.8+
- OpenSfM (up to date version from GitHub)
- NumPy
- OpenCV

Key toolset features:

- Import and export datasets between SQLite and OpenSfM formats.
- Import and export datasets between SQLite and COLMAP formats.
- Pluggable interface for custom image matching models.
