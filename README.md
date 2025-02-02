[![Docker Pulls](https://img.shields.io/docker/pulls/flywheel/dicom-send.svg)](https://hub.docker.com/r/flywheel/dicom-send/)
[![Docker Stars](https://img.shields.io/docker/stars/flywheel/dicom-send.svg)](https://hub.docker.com/r/flywheel/dicom-send/)

# dicom-send Gear

A [Flywheel Gear](https://github.com/flywheel-io/gears/tree/master/spec) to execute DCMTK's [storescu](https://support.dcmtk.org/docs/storescu.html) to send DICOM data from a [Flywheel](flywheel.io) instance to a DICOM server.


## Description

The [DICOM Toolkit](https://support.dcmtk.org/docs/) (DCMTK) is a set of software libraries for working with the DICOM Standard. In particular, the [storescu](https://support.dcmtk.org/docs/storescu.html) application is useful for transmitting DICOM images. The dicom-send Gear uses DCMTK's storescu to send DICOM data from a [Flywheel](flywheel.io) instance to a specific DICOM server. The DICOM server must be reachable from the host of the [Flywheel](flywheel.io) instance. Before transmitting the DICOM file, a private tag indicating the DICOM file's source as Flywheel is added to each DICOM file to avoid being re-ingested into a [Flywheel](flywheel.io) instance.


### Gear Inputs

* **file**: Any DICOM file or an archive (zip or tar) containing DICOM file(s). Non DICOM files are ignored. If no input is provided, all DICOM files in the session where the Gear is executed are downloaded and used as input.
* **api_key**: Gear will acquire the read-only API key for downloading DICOMs from the session when no input **file** is provided.


### Configuration Settings

* **destination**: The IP address or hostname of the destination DICOM server. Note: The server must be reachable from the host of the Flywheel instance.
* **called_ae**: The Called AE title of the receiving DICOM server.
* **calling_ae**: The Calling AE title. Default = flywheel.
* **port**: Port number of the listening DICOM service. Default = 104.

### Gear outputs

The gear will generate a report listing each dicom file/archive that was exported.  The report includes the following columns:
* ***Acquisition ID:*** the FW Acquisition ID
* ***FW Path:*** a human readable flywheel path to the file that was exported, in the following format:
    * <group>/<project.label>/<subject.label>/<session.label>/<acquisition.label>/files/<file.name>
* ***Filename:*** Name of the file/archive that was sent
* ***Images in Series:*** Number of images in the series to be sent
* ***Images Sent:*** Number of images successfully sent
* ***Status:*** “Complete” if images in series == Images Sent, “Incomplete” if Images Sent < Images in Series, “Failed” if Images Sent == 0.

This report is printed at the end of the log file, and also saved as an attachment to the session container that the gear was run from.
The output name of this report file follows the following pattern:

`dicom-send_report-<session label>_<acquisition label>_YYYY-MM-DD_HH:MM:SS.csv`
where `<acquisition_label>` is only present if one specific acquisition was selected for export.

Images are considered successfully sent if `storescu` returns a "0" exit code.

The gear will present as successful only if all dicoms that were attempted, were sent succesfully.


# Workflow

1. *Acquire Inputs*. If the **file** input is not provided, all files of DICOM type in the session where the Gear is executed are downloaded and set as input for the next stage.
2. *Prepare Inputs*. DICOMs packaged into archives (.tgz or .zip) are uncompressed. A private tag is then added to each DICOM file to be transmitted.
3. *Transmit DICOMs*. The final stage transmits each DICOM file to the DICOM server indicated during Gear configuration.

# Testing

For information on gear testing, see the [testing readme](TESTING.md).
