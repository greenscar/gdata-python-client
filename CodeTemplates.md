# Introduction #

This page is a resource for anyone planning to write a sample app or series of unit tests which interact with a Google Data service.

# Data Model Unit Test #

Data model tests cover code written in the gdata.x.data modules and they should not include any communication with the external server. The focus is on correct formatting and preservation of information in the XML parsing/generating code.

```
#!/usr/bin/env python
#
# Copyright (C) <year> Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import unittest
import gdata.test_config as conf
<imports>


class XTest(unittest.TestCase):

  def test_something_here(self):
    ...


class YTest(unittest.TestCase):
  ...


def suite():
  return conf.build_suite([XTest, YTest, ...])


if __name__ == '__main__':
  unittest.main()
```


# Service Client Unit Test #

Client tests are designed to make live requests to the server and verify correctness of the code in the gdata.x.client module. Because live HTTP requests to the server can be time consuming, the test configuration module provides several time saving options such as caching and replaying server responses from a previous test session run and allowing tests which require server contact to be skipped entirely. For this reason the client unit tests require additional setUp and tearDown code, as well as a recommended few lines in each test method. Begin with the template below as an example to take advantage of the time saving configuration features.

```
#!/usr/bin/env python
#
# Copyright (C) <year> Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import unittest
import gdata.test_config as conf
<imports>


class XTest(unittest.TestCase):

  def setUp(self):
    self.client = None
    if conf.options.get_value('runlive') == 'true':
      self.client = <desired_client_class>() # Ex: gdata.client.GDClient()
      # Pass in the client oject, name of the TestCase as a string, and the
      # service name as used in ClientLogin, for example 'blogger', 'wise',
      # 'cp', 'local', etc.
      conf.configure_client(self.client, 'XTest', 'service name') 

  def tearDown(self):
    conf.close_client(self.client)

  def test_xyz(self):
    if not conf.options.get_value('runlive') == 'true':
      return
    # Either load the recording or prepare to make a live request.
    conf.configure_cache(self.client, 'test_xyz')
    ...


def suite():
  return conf.build_suite([XTest, ...])


if __name__ == '__main__':
  unittest.TextTestRunner().run(suite())
```

# Command Line Sample #

# App Engine Sample App #

# Data Model Module #

This library uses Python classes to represent XML elements which are specific to each Google Data service. For more information on how these classes work, see the WritingDataModelClasses wiki page.

```
#!/usr/bin/env python
#
# Copyright (C) 2009 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


"""Data model classes for parsing and generating XML for the X API."""


__author__ = 'you@example.com (Your Name)'


import atom.core
import gdata.data
<other imports>


# XML namespace templates and constants.
# The below examples are for Blogger.
LABEL_SCHEME = 'http://www.blogger.com/atom/ns#'
THR_TEMPLATE = '{http://purl.org/syndication/thread/1.0}%s'

class BlogPost(gdata.data.GDEntry):
  """Represents a single post on a blog."""

  # Define any convenience methods which make the class easier to use.
  def add_label(self, label):
    """Adds a label to the blog post.

    The label is represented by an Atom category element, so this method
    is shorthand for appending a new atom.Category object.

    Args:
      label: str 
    """
    self.category.append(atom.data.Category(scheme=LABEL_SCHEME, term=label))

  AddLabel = add_label


class BlogPostFeed(gdata.data.GDFeed):
  entry = [BlogPost]


class InReplyTo(atom.core.XmlElement):
  _qname = THR_TEMPLATE % 'in-reply-to'
  href = 'href'
  ref = 'ref'
  source = 'source'
  type = 'type'


class Comment(BloggerEntry):
  """Blog post comment entry in a feed listing comments on a post or blog."""
  in_reply_to = InReplyTo

  def get_comment_id(self):
    """Extracts the commentID string from the entry's Atom id.

    Returns: A string of digits which identify this post within the blog.
    """
    if self.id.text:
      return COMMENT_ID_PATTERN.match(self.id.text).group(1)
    return None

  GetCommentId = get_comment_id


class CommentFeed(gdata.data.GDFeed):
  entry = [Comment]
```

# Service Client Module #