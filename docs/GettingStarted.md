# Getting Started

1. Create a notebook in the [Wolfram Cloud](https://www.wolframcloud.com/) and make it public.
    * In the Cloud UI, use the "New Notebook" button and then use the "Share" dialog to make it public.
    * From the Wolfram Language, you could deploy notebook content like so:

            CloudDeploy[Manipulate[Plot[Sin[a*x], {x, 0, 2Pi}], {a, 1, 3}], Permissions->{All->{"Read", "Interact"}}]
2. Install this library in your JS project using

        npm install wolfram-notebook-embedder
        
    so you can import it in JS as
    
        import WolframNotebookEmbedder from 'wolfram-notebook-embedder';
        
    or import it as a `<script>` tag from a CDN:
    
        <script crossorigin src="https://unpkg.com/wolfram-notebook-embedder@0.1/dist/wolfram-notebook-embedder.min.js"></script>
3. In your HTML, create a container where you want the notebook to be rendered (let's say `<div id="notebookContainer"></div>`) and add the following JS code:

        var embedding = WolframNotebookEmbedder.embed('url', document.getElementById('notebookContainer'));
    
    where `url` is the URL of your cloud object from step 1. More details about `embed` are described in the [library interface documentation](./LibraryInterface.md).
4. If you want to control the notebook from your JS code, wait for the `embed` result (a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)) to resolve and then use various [Notebook API methods](./NotebookAPI.md):

        embedding.then(function (nb) {
            // This will reset the DynamicModule variable x$$
            // in the first cell of the notebook.
            return nb.getCells().then(function (cells) {
                nb.setDynamicModuleVariable({
                    cellId: cells[0].id,
                    name: 'x$$',
                    value: 0
                });
            });
        })
5. If you want to serve static HTML from your server so the notebook can be rendered before JS code is loaded (which also helps with SEO), take a look at [server-side rendering](./ServerSideRendering.md).
6. If you run into any issues, take a look at the [Troubleshooting guide](./Troubleshooting.md). If you think you found a bug, please report it.