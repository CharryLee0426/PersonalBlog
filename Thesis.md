## 1. FrontEnd Dependencies
* react-pdf-viewer;
* react-file-viewer;

example code:

```javascript
import * as React from 'react';
import { Worker } from '@react-pdf-viewer/core';
import { Viewer } from '@react-pdf-viewer/core';
import '@react-pdf-viewer/core/lib/styles/index.css';
import FileViewer from 'react-file-viewer';
import { Button } from '@mui/material';

export default function Test() {
    return (
        <Worker workerUrl="https://unpkg.com/pdfjs-dist@3.4.120/build/pdf.worker.min.js">
            <div style={{ border: '1px solid rgba(0, 0, 0, 0.3)', }}>
                <Viewer fileUrl={PDF}/>
            </div>
        </Worker>
    );
}

// export default function Test() {
//     const file = './blank.docx';
//     const type = 'docx';

//     return(
//         <FileViewer
//         fileType={type}
//         filePath={DOC}
//         />
//     );
// }
```