<a href="https://www.crownpeak.com" target="_blank">![Crownpeak Logo](images/crownpeak-logo.png?raw=true "Crownpeak Logo")</a>

# Crownpeak Digital Experience Management (DXM) Software Development Kit (SDK) for React
Crownpeak Digital Experience Management (DXM) Software Development Kit (SDK) for React has been constructed to assist
the Single Page App developer in developing client-side applications that leverage DXM for content management purposes.

## Benefits
* **Runtime libraries to handle communication with either Dynamic (DXM Dynamic Content API) or Static (On-disk JSON payload)
Data Sources**

  As a development team runs their build process, the underlying React Application will be minified and likely packed
  into a set of browser-compatible libraries (e.g., ES5). We expect any DXM NPM Packages also to be compressed in this
  manner. To facilitate communication between the React Application and content managed within DXM, a runtime NPM Package
  is provided. The purpose of this package is:
  
  * Read application configuration detail from a global environment file (e.g., Dynamic Content API endpoint and credentials, 
  static content disk location, etc.);
  * Making data models available to the React Application, which a developer can map against
    * **Dynamic Data** - Asynchronously processing data from the DXM Dynamic Content API, using the Search G2 Raw JSON endpoint; and 
    * **Static Data** - Loading JSON payload data directly from local storage.
  
* **DXM Content-Type Scaffolding**

  Developers will continue to work with their Continuous Integration / Delivery and source control tooling to create a
  React application. However, the purpose of the DXM Content-Type Scaffolding build step is to convert the React Components
  in a single direction (React > DXM), into the necessary configuration to support CMS operations. At present, the DXM
  Component Library includes the capability to auto-generate Templates (input.aspx, output.aspx, post_input.aspx) based
  upon a moustache-style syntax (decorating of editable properties). It is not intended that we re-design this process,
  as it is fully supported within DXM, and customer-battle-tested - therefore, in order to create Template configuration,
  the build step:
    * Converts React Components into Crownpeak Components by using the existing Component Builder Process, via the CMS Access
   API (DXM's RESTful Content Manipulation API), and then existing "post_save" process;
    * Creates Templates for each React Page (One of the DXM React Component Types) by using the existing Template Builder
   Process, again via the CMS Access API and existing "post_save" process; and
    * Creates a new Model for the React Page Content-Type, via the CMS Access API, so that authors can create multiple versions
   of a structured Page or Component, without needing to run an entire development/test cycle.
   
## Install
 ```
npm install react-html-parser
# or 
yarn add react-html-parser
```
 
## Usage - Runtime Data Libraries
 Review example project at <a href="https://github.com/Crownpeak/DXM-React-SDK/tree/master/examples/bootstrap" target="_blank">https://github.com/Crownpeak/DXM-React-SDK/tree/master/examples/bootstrap</a>
 for complete usage options. The example project includes the following capabilities:
  * Routing using ```react-router``` and JSON payload, delivered from DXM to map AssetId to requested path;
  * ```CmsStaticPage``` type to load payload data from JSON file on filesystem, delivered by DXM;
  * ```CmsDynamicPage``` type to load payload data from DXM Dynamic Content API.
 
### CmsStaticPage Type
Loads payload data from JSON file on filesystem - expects knowledge of DXM AssetId in order to find file with corresponding
name (e.g., 12345.json). CmsStaticPage is the data equivalent of a DXM Asset when used as a page. Example at /examples/bootstrap/pages/blogPage.js:
```
import React from 'react'
import BlogPost from "../components/blogPost";
import { CmsStaticPage } from 'crownpeak-dxm-react-sdk';

export default class BlogPage extends CmsStaticPage
{
    render() {
        super.render();
        return (
                <div className="container">
                    <BlogPost/>
                </div>
            </div>
        )
    }
}
```

### CmsDynamicPage Type
Loads payload data from DXM Dynamic Content API upon request - expects knowledge of DXM AssetId. Example at /examples/bootstrap/pages/blogPage.js:
 ```
import React from 'react'
import BlogPost from "../components/blogPost";
import { CmsDynamicPage } from 'crownpeak-dxm-react-sdk';

export default class BlogPage extends CmsDynamicPage
{
    render() {
        super.render();
        return (
                <div className="container">
                    <BlogPost/>
                </div>
            </div>
        )
    }
}
```

### CmsComponent
Includes CmsField references for content rendering from DXM within a React Component. Example at /examples/bootstrap/components/blogPost.js:
```
import React from 'react';
import { CmsComponent, CmsField, CmsFieldTypes } from 'crownpeak-dxm-react-sdk';
import ReactHtmlParser from 'react-html-parser';

export default class BlogPost extends CmsComponent
{
    constructor(props)
    {
        super(props);
        this.post_title = new CmsField("Post_Title", CmsFieldTypes.TEXT);
        this.post_date = new CmsField("Post_Date", CmsFieldTypes.DATE);
        this.post_content = new CmsField("Post_Content", CmsFieldTypes.WYSIWYG);
        this.post_category = new CmsField("Post_Category", CmsFieldTypes.DOCUMENT);
    }

    render() {
        return (
            <div className="blog-post">
                <h2 className="blog-post-title">{ this.post_title }</h2>
                <p className="blog-post-meta">Date: { new Date(this.post_date).toLocaleDateString() }</p>
                { ReactHtmlParser(this.post_content) }
                { /*this.post_category*/ }
            </div>
        )
    }
}
```

### CmsFieldType
Enumeration containing field types supported within the SDK.

| CmsFieldType  | DXM Mapping   |
| ------------- |---------------|
| TEXT          | Text          |
| WYSIWYG       | Wysiwyg       |
| DATE          | DateTime      |
| DOCUMENT      | Document      |
| IMAGE         | Image         |


### Querying Custom Data from Dynamic Content API
Used to run a one-time dynamic query from DXM's Dynamic Content API. Example at /examples/bootstrap/components/postArchives.js:
```
import React from 'react';
import { Link } from 'react-router-dom';
import { CmsComponent, CmsDynamicDataProvider } from 'crownpeak-dxm-react-sdk';

export default class PostArchives extends CmsComponent
{
    constructor(props)
    {
        super (props);
        const data = CmsDynamicDataProvider.getDynamicQuery("q=*:*&fq=custom_s_type:\"Blog%20Page\"&rows=0&facet=true&facet.mincount=1&facet.range=custom_dt_created&f.custom_dt_created.facet.range.start=NOW/YEAR-1YEAR&f.custom_dt_created.facet.range.end=NOW/YEAR%2B1YEAR&f.custom_dt_created.facet.range.gap=%2B1MONTH");
        this.months = data.facet_counts.facet_ranges.custom_dt_created.counts.filter((_c, i) => i%2 === 0);
    }

    render() {
        return (
            <div className="p-3">
                <h4 className="font-italic">Archives</h4>
                <ol className="list-unstyled mb-0">
                    {this.months.map((month) => {
                        return <li key={month.substr(0,7)}><Link to={`/posts/months/${month.substr(0,7)}`}>{ [new Date(month).toLocaleString('default', { month: 'long', year: 'numeric' })] }</Link></li>
                    })}
                </ol>
            </div>
        )
    }
}
```

### Using Custom Data from Named JSON Object on Filesystem
Used to load content from a JSON Object on Filesystem and populate fields in CmsComponent. Example at /examples/bootstrap/components/topicList.js:
```
import React from 'react';
import { CmsComponent, CmsStaticDataProvider } from 'crownpeak-dxm-react-sdk';

export default class TopicList extends CmsComponent
{
    constructor(props)
    {
        super (props);
        this.topics = CmsStaticDataProvider.getCustomData("topics.json");
    }

    render() {
        return (
            <div className="nav-scroller py-1 mb-2">
                <nav className="nav d-flex justify-content-between">
                    {this.topics.map((topic) => {
                        return <a key={topic.toString()} className="p-2 text-muted" href="#">{ topic }</a>
                    })}
                </nav>
            </div>
        )
    }
}
```