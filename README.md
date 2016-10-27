# Campbell Scientific Datalogger Parser
Utility for parsing and exporting data outputted by Campbell Scientific, Inc. CR-type dataloggers.
## Features
* Reads data generated by various CR-type dataloggers into a manageable data structure (lists of ordered dictionaries).
* Supports both table and mixed array data.
* Ability to extract certain column data on specific timestamps.
* Provides time parsing with the ability to convert different CR-type time representations into Python datetime objects.
* Ability to export processed data to CSV.

## Installation
Source code hosted on GitHub at https://github.com/SunBurst/campbellsciparser

```sh
# Install using pip
pip install campbellsciparser
```

## Examples
```sh
>>> from campbellsciparser import cr 
```
To read table data
```sh
>>> data_table = read_table_data('/path/to/table_data.dat', header_row=0)
>>> data_table
[OrderedDict([('Time', '2016-06-01 12:00:00'), ('Air_Temperature', '11.464')]), 
OrderedDict([('Time', '2016-06-02 12:00:00'), ('Air_Temperature', '12.320')]),
OrderedDict([('Time', '2016-06-03 12:00:00'), ('Air_Temperature', '12.555')]),
OrderedDict([('Time', '2016-06-04 12:00:00'), ('Air_Temperature', '11.639')]),
OrderedDict([('Time', '2016-06-05 12:00:00'), ('Air_Temperature', '10.564')])]
```
To read specific rows
```sh
>>> data_table = read_table_data('/path/to/table_data.dat', header_row=0, first_line_num=2, last_line_num=3)
>>> data_table
OrderedDict([('Time', '2016-06-03 12:00:00'), ('Air_Temperature', '12.555')]),
OrderedDict([('Time', '2016-06-04 12:00:00'), ('Air_Temperature', '11.639')]),
```
Using custom header
```sh
>>> data_table = read_table_data('/path/to/table_data.dat', 
... header=['Custom_Time_Column_Name', 'Custom_Data_Column_Name'], first_line_num=1)
>>> data_table[0]
OrderedDict([('Custom_Time_Column_Name', '2016-06-01 12:00:00'), ('Custom_Data_Column_Name', '11.464')])
```
Using no header (assigns column indices as names)
```sh
>>> data_table = read_table_data('/path/to/table_data.dat', first_line_num=1)
>>> data_table[0]
OrderedDict([(0, '2016-06-01 12:00:00'), (1, '11.464')])
```
To read mixed array data
```sh
>>> data_mixed_array = read_mixed_array_data('/path/to/mixed_array_data.dat')
>>> data_mixed_array
[OrderedDict([(0, '100'), (1, '2016'), (2, '159'), (3, '0'), (4, '11.273')]), 
OrderedDict([(0, '101'), (1, '2016'), (2, '159'), (3, '17.320')]
OrderedDict([(0, '110'), (1, '2016'), (2, '159'), (3, '0'), (4, '11.379')]),
OrderedDict([(0, '110'), (1, '2016'), (2, '159'), (3, '5'), (4, '11.443')]),
OrderedDict([(0, '110'), (1, '2016'), (2, '159'), (3, '10'), (4, '11.407')]),
OrderedDict([(0, '110'), (1, '2016'), (2, '159'), (3, '15'), (4, '11.340')])]
```
Filter mixed array data by array id
```sh
>>> filter_mixed_array_data(data, '100')
defaultdict(<class 'list'>, {'100': [OrderedDict([(0, '100'), (1, '2016'), 
(2, '159'), (3, '0'), (4, '11.273')])]})
```
Read array ids data
```sh
>>> data_by_array_ids = read_array_ids_data('/path/to/mixed_array_data.dat', 
... array_id_names={'100': 'Hourly'})
>>> hourly_data = data_by_array_ids.get('Hourly')
>>> hourly_data
[OrderedDict([(0, '100'), (1, '2016'), (2, '159'), (3, '0'), (4, '11.273')])]
```
Parsing time
```sh
>>> hourly_data_localized = parse_time(data=hourly_data, time_zone='Europe/Stockholm',
... time_format_args_library=['%Y', '%j', '%H%M'], time_columns=[1, 2, 3])
>>> hourly_data_localized
[OrderedDict([(0, '100'), (1, datetime.datetime(2016, 6, 7, 0, 0, 
tzinfo=<DstTzInfo 'Europe/Stockholm' SET+1:00:00 STD>))), (4, '11.273')])]
```
Converting time zones
```sh
>>> hourly_data_as_utc = convert_time_zone(data=hourly_data_localized, time_column=1, time_zone='UTC')
>>> hourly_data_as_utc
[OrderedDict([(0, '100'), (1, datetime.datetime(2016, 6, 6, 23, 0, tzinfo=<UTC>)), (4, '11.273')])]
```
Read and parse time table data in one call
```sh
>>> data_table = read_table_data(
... '/path/to/table_data.dat',
... header_row=0,
... parse_time=True,
... time_zone='Europe/Stockholm',
... time_format_args_library=['%Y-%m-%d %H:%M:%S'],
... time_parsed_column='TIMESTAMP',
... time_columns=['Time']
... )
>>> data_table[0]
OrderedDict([('Timestamp', datetime.datetime(2016, 6, 1, 12, 0, tzinfo=<DstTzInfo 'Europe/Stockholm' 
CEST+2:00:00 DST>), ('Air_Temperature', '11.464')])
```
Extract columns data
```sh
>>> extract_columns_data(data_table, 'Air_Temperature')[:3]
[OrderedDict([('Air_Temperature', '11.464')]), OrderedDict([('Air_Temperature', '12.320')]),
OrderedDict([('Air_Temperature', '12.555')])]

>>> extract_columns_data(data_table, 'Timestamp', 'Air_Temperature', time_column='Timestamp',
... from_timestamp=datetime(2016, 6, 3, 12, 0, 0, tzinfo=pytz.UTC))
[OrderedDict([('Time', datetime.datetime(2016, 6, 3, 12, 0, tzinfo=<UTC>), ('Air_Temperature', '12.555')]),
OrderedDict([('Time', datetime.datetime(2016, 6, 4, 12, 0, tzinfo=<UTC>)), ('Air_Temperature', '11.639')]),
OrderedDict([('Time', datetime.datetime(2016, 6, 5, 12, 0, tzinfo=<UTC>), ('Air_Temperature', '10.564')])]

>>> extract_columns_data(data_table, 'Timestamp', 'Air_Temperature', time_column='Timestamp',
... to_timestamp=datetime(2016, 6, 3, 12, 0, 0, tzinfo=pytz.UTC))
[OrderedDict([('Time', datetime.datetime(2016, 6, 1, 12, 0, tzinfo=<UTC>)), ('Air_Temperature', '11.464')]), 
OrderedDict([('Time', datetime.datetime(2016, 6, 2, 12, 0, tzinfo=<UTC>)), ('Air_Temperature', '12.320')]),
OrderedDict([('Time', datetime.datetime(2016, 6, 3, 12, 0, tzinfo=<UTC>), ('Air_Temperature', '12.555')])]

>>> extract_columns_data(data_table, 'Timestamp', 'Air_Temperature', time_column='Timestamp',
... from_timestamp=datetime(2016, 6, 3, 12, 0, 0, tzinfo=pytz.UTC), 
... to_timestamp=datetime(2016, 6, 3, 12, 0, 0, tzinfo=pytz.UTC))
[OrderedDict([('Time', datetime.datetime(2016, 6, 3, 12, 0, tzinfo=<UTC>), ('Air_Temperature', '12.555')])]
```
Update column names
```sh
>>> new_column_names = ['New_Label_1', 'New_Label_2']
>>> new_column_names_result = update_column_names(data_table, column_names=new_column_names)
>>> new_column_names_result[:3]
[OrderedDict([('New_Label_1', datetime.datetime(2016, 6, 1, 12, 0, tzinfo=<UTC>)), ('New_Label_2', '11.464')]), 
OrderedDict([('New_Label_1', datetime.datetime(2016, 6, 2, 12, 0, tzinfo=<UTC>)), ('New_Label_2', '12.320')]),
OrderedDict([('New_Label_1', datetime.datetime(2016, 6, 3, 12, 0, tzinfo=<UTC>), ('New_Label_2', '12.555')])]
```
Export to CSV
```sh
>>> export_to_csv(data_table, 'path/to/output_file_w_header.dat', export_header=True)
>>> exported_data = read_table_data('path/to/output_file_w_header.dat', header_row=0)
>>> exported_data[0]
[OrderedDict([('Time', '2016-06-01 12:00:00'), ('Air_Temperature', '11.464')])]

>>> export_to_csv(data_table, 'path/to/output_file_wo_header.dat', export_header=False)
>>> exported_data = read_table_data('path/to/output_file_wo_header.dat')
>>> exported_data[0]
[OrderedDict([(0, '2016-06-01 12:00:00'), (1, '11.464')])]

>>> export_to_csv(data_table, 'path/to/output_file_w_tz.dat', export_header=True, include_time_zone=True)
>>> exported_data = read_table_data('path/to/output_file_w_tz.dat', header_row=0)
>>> exported_data[0]
[OrderedDict([(0, '2016-06-01 12:00:00+0000'), (1, '11.464')])]
```
Export array ids data to CSV
```sh
>>> array_ids_data = {
... '100': [OrderedDict([(0, '100'), (1, '2016'), (2, '159'), (3, '0'), (4, '11.273')])],
... '101': [OrderedDict([(0, '101'), (1, '2016'), (2, '159'), (3, '17.320')])]
...}
>>> array_ids_info = {
... '100': {'file_path': 'path/to/outputfile_100.dat'}, 
... '101': {'file_path': 'path/to/outputfile_101.dat'}
... }
>>> export_array_ids_to_csv(array_ids_data, array_ids_info, export_header=True)
>>> exported_data_100 = read_table_data('path/to/outputfile_100.dat', header_row=0)
>>> exported_data_101 = read_table_data('path/to/outputfile_101.dat', header_row=0)
>>> exported_data_100
[OrderedDict([(0, '100'), (1, '2016'), (2, '159'), (3, '0'), (4, '11.273')]
>>> exported_data_101
[OrderedDict([(0, '101'), (1, '2016'), (2, '159'), (3, '17.320')])]
```

## Dependencies
* pytz

## Documentation
Documentation at https://readthedocs.org/projects/campbellsciparser/
