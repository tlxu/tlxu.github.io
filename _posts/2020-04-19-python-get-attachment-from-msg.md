---
layout: post
title:  "Python to Retrieve attachment from saved outlook msg file"
date:   2020-04-19
categories: Python, Data
---

The code snippet below:
1. Finds all outlook msg files in directory ./cibc_mello_ald/ recursively,
2. Retrieves the attached .csv file in each .msg file,
3. Then saves attachments to directory ./cibc_mello_ald/csv/

**Note** that to make it work, you need to:
- Run it in a Windows machine
- With Outlook installed
- Use absolute path when accessing files
- Close(or Open and then close) Outlook if you see an error complaining the msg file is already open or you don't have permision to open it.

``` python
for idx, file in enumerate(msg_files):
    # get file date 02-Apr-2020
    print(f'{idx+1}/{number_of_files}: {file}')
    date_str = re.split(' |\.', file)[-2]
    print(date_str)

    # Read attachments
    outlook = win32com.client.Dispatch('Outlook.Application').GetNamespace('MAPI')
    msg = outlook.OpenSharedItem(file)
    att = msg.Attachments
    for i in att:
        csv_file_name = os.path.join(csv_file_dir, f'{csv_file_prefix}-{date_str}.csv')
        i.SaveAsFile(csv_file_name)
```

[source code](https://github.com/tlxu/Quanance/blob/master/cibc_mellon_ald/python_get_msg_attachment.py)
