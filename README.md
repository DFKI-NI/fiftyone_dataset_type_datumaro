# Fiftyone dataset type 'DatumaroDataset'
The implementation defines a new dataset type 'DatumaroDataset' with a custom DatumaroDatasetImporter and -Exporter. \
It currently supports only images as media types and Label, Mask, Polygon, Bbox and Points as annotation types. \
Further details regarding the datumaro format can be found in the <a href="https://open-edge-platform.github.io/datumaro/stable/docs/data-formats/formats/datumaro.html">datumaro documentation</a>. 

The 'DatumaroDataset' type implementation supports almost all features and functions as the natively provided 'COCODetectionDataset' dataset type. Furthermore the 'DatumaroDataset' type importer and exporter functions provide several additional features for a more advanced dataset management. Some of the additional features are:
- read metadata informations stored as tEXt chunks from .png-files into fiftyone fields
- option to create label tags from label attributes
- export of several label fields together into one .json-file


# Installation
### 1. Install the python requirements:
```bash
pip3 install -r python_requirements.txt
```

### 2. Copy the datumaro.py-file from this repository to the site-packages path of your installed fiftyone instance utils folder, e.g.:
```bash
cp datumaro.py ~/.local/lib/python3.12/site-packages/fiftyone/utils/
```

### 3. Add the following lines at the end of your ../site-packages/fiftyone/types/dataset_types.py file:
```python
class DatumaroDataset(LabeledImageDataset):
    """A labeled dataset consisting of images and their associated annotations saved in
    `Datumaro Format <https://open-edge-platform.github.io/datumaro/latest/docs/data-formats/formats/datumaro.html>`_.
    """

    def get_dataset_importer_cls(self):
        import fiftyone.utils.datumaro as foud

        return foud.DatumaroDatasetImporter

    def get_dataset_exporter_cls(self):
        import fiftyone.utils.datumaro as foud

        return foud.DatumaroDatasetExporter
```

### 4. Restart your fiftyone instance


# Usage
The Importer expects the dataset to be in the following folder structure (only the 'default' subset prefix is supported so far):
```
image_datasets/
├── annotations/
│   └── default.json
└── images/
    └── default/
        |── <image file as .png>
        └── ...
```

### Example python usage:
```python
import fiftyone as fo
from fiftyone.utils.data import importers as fud_importers
from fiftyone.utils.data import exporters as fud_exporters
from fiftyone.utils import datumaro as fu_datumaro
import os


dataset_name = ""
dataset_dir = ""
label_json = ""
export_dir = ""

## dataset creation and import
dataset = fo.Dataset.from_dir(
    dataset_dir=dataset_dir,
    dataset_type=fo.types.DatumaroDataset,
    name=dataset_name,
    overwrite=True,
    dynamic=True,
    persistent=True,
    label_field="ground_truth",
    load_all_images_from_data_path=True,
    read_metadata_from_file=False
)

## import of images into an existing dataset
data_importer = fu_datumaro.DatumaroDatasetImporter(dataset_dir=dataset_dir,
                                                    read_metadata_from_file=True)
fud_importers.import_samples(dataset=dataset,
                             dataset_importer=data_importer,
                             dynamic=True)

## export dataset
data_exporter = fu_datumaro.DatumaroDatasetExporter(export_dir=export_dir,
                                                    export_media=True)
fud_exporters.export_samples(dataset,
                             dataset_exporter=data_exporter)

## add label to existing images in a dataset
fu_datumaro.add_datumaro_labels(dataset,
                                label_field="ground_truth",
                                labels_or_path=label_json)
```