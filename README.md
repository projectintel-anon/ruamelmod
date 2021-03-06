ruamel.std.encryptedzip is a modified library that adds support for encrypted zip archives for Python to ruamel.std.zipfile; linked below.

https://pypi.org/project/ruamel.std.zipfile/

It is super simple, it just adds an extra parameter to the API:

```py
    def delete_from_zip_file(self, pattern=None, file_names=None, password=None): #Take extra parameter for password.
        """
        zip_file can be a string or a zipfile.ZipFile object, the latter will be closed
        any name in file_names is deleted, all file_names provided have to be in the ZIP
        archive or else an IOError is raised
        """
        if pattern and isinstance(pattern, string_type):
            import re
            pattern = re.compile(pattern)
        if file_names:
            if not isinstance(file_names, list):
                file_names = [str(file_names)]
            else:
                file_names = [str(f) for f in file_names]
        else:
            file_names = []
        with zipfile.ZipFile(self._file_name) as zf:
            zf.setpassword(password) #Set the password for the zip file.
            for l in zf.infolist():
                if l.filename in file_names:
                    file_names.remove(l.filename)
                    continue
                if pattern and pattern.match(l.filename):
                    continue
                self.append(l.filename, zf.read(l))
            if file_names:
                raise IOError('[Errno 2] No such file{}: {}'.format(
                    '' if len(file_names) == 1 else 's',
                    ', '.join([repr(f) for f in file_names])))


def delete_from_zip_file(file_name, pattern=None, file_names=None, password=None): #Take extra parameter for password (overloaded function).
    with InMemoryZipFile(file_name) as imz:
        imz.delete_from_zip_file(pattern, file_names, password) #Parse the extra parameter to the class' method.
```
It may look scary, but it is very simple. Instead of calling the original function with just a pattern or file_names array, you add an extra parameter where needed:

```py
from ruamelmod import delete_from_zip_file
#Code for zip file stuff goes here
password_zip = b'testing123'
delete_from_zip_file("test.zip", file_names=['test.txt', 'test2.txt'], password=password_zip)
```

If I remember correctly, Python will throw a TypeError if your password is not encoded into UTF-8 bytes. At the moment, there is no support for directly giving the method your zipfile.ZipFile object, but I will hopefully add support for that in the future.

All credit for code goes to Anthon at Ruamel, all I did was add in a few words and things here and there to add some extra support.
