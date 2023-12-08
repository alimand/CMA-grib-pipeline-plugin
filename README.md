# grib2-pipeline-plugin
grib2-pipeline-plugin

# 1.	Related File
a.universal.py

b.data-mappings.yml

# 2.	Source Code
"""create function: UniversalData，inherit wis2box.data.base.BaseAbstractData"""

Implement the transform method and fill in the output_data property, returning True
# universal.py
 	from datetime import datetime
 	import logging
 	from pathlib import Path
 	import re
 	from typing import Union
 	
 	from dateutil.parser import parse
 	
 	from wis2box.data.base import BaseAbstractData
 	
 	LOGGER = logging.getLogger(__name__)
 	
 	
 	class UniversalData(BaseAbstractData):
 	    """Universal data"""
 	
 	    def __init__(self, defs: dict) -> None:
 	        super().__init__(defs)
 	
 	    def transform(self, input_data: Union[Path, bytes],
 	                  filename: str = '') -> bool:
 	
 	        filename2 = Path(filename)
 	        LOGGER.debug('Procesing data')
 	        input_bytes = self.as_bytes(input_data)
 	
 	        LOGGER.debug('Deriving datetime')
 	
 	        match = re.search(self.file_filter, filename2.name)
 	        if match:
 	            date_time = match.group(1)
 	        else:
 	            LOGGER.debug('Could not derive date/time: using today\'s date')
 	            date_time = datetime.now()
 	
 	        if date_time:
 	            date_time = parse(date_time)
 	
 	        rmk = filename2.stem
 	        suffix = filename2.suffix.replace('.', '')
 	
 	        self.output_data[rmk] = {
 	            suffix: input_bytes,
 	            '_meta': {
 	                'identifier': rmk,
 	                'relative_filepath': self.get_local_filepath(date_time),
 	                'data_date': date_time
 	            }
 	        }
 	
 	        return True
 	
 	    def get_local_filepath(self, date_):
 	        yyyymmdd = date_.strftime('%Y-%m-%d')
 	        return Path(yyyymmdd) / 'wis' / self.topic_hierarchy.dirpath
# 3.	Data-mappings.yml configures the topic hierarchy of the numerical prediction data (CMA as an example)
# data-mappings.yml
 	data: 		
    chn.babj.data.core.weather.prediction.forecast.shortrange.probabilistic.global.CMA_GEPS::
        plugins:
            grib2:
 	"""call grib2 data pipeline plugin to deal with CMA_GEPS grib2 data"""
                - plugin: wis2box.data.universal.UniversalData
                  notify: true
                  buckets:
                    - ${WIS2BOX_STORAGE_INCOMING}
                  file-pattern: '^.*_(\d{8})\d{2}.*\.grib2$'
# 4.	Test data list
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-024.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-036.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-048.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-060.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-072.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-084.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-096.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-108.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-120.grib2
 	Z_NAFP_C_BABJ_20231207000000_P_CMA-GEPS-GLB-132.grib2
