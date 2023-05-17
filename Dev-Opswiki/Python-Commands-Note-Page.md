DO NOT FORGET DOCSTRINGS!!!!! """information about YOUR CODE"""

Strings notes:
how to uppercase - something_nothing = something_nothing.upper()

how to tie in OBJ into STR 
- something_nothing1 = something_nothing[INT]
- something_nothing2 = something_nothing[INT:INT]
- something_nothing3 = something_nothing[INT:]

return f'{something_nothing1}_{something_nothing2}_{something_nothing3}'

Lists notes:
How to make your list
- something1 = []

modifying your list using a for loop

- for something in somethings:
- if something[-3:] == 'OBJ':
- something1.append(something)

return something
~repeat~

- nothing = []
- for something in somethings:
- if something[-4:] == 'OBJE'
- nothing.append(something)

return nothing

Dictionary notes:

making / modifying a dictionary
- name = {}
- name['item1'] = default_name
- name['item2'] = SOMETHING[default_name[0]]
- name['item3'] = default_name.split('_')[1]
- default_type_abbr = default_name.split('_')[2]
- name['item4'] = DEFAULT_TYPES[default_type_abbr]

return name

Nested Data notes:
modifying nested data / finding information
- something_nothing = []
- for key,value in nothing_data.items():
- if v['network'] == 'red':
- for interface in v['interfaces']:
- if interface['ip_address'] in ['random ip', 'random ip']:
- something_nothing.append(k)

return something_nothing
