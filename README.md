<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Model Testing Platform</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/STLLoader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        .model-container { border: 1px solid #ddd; padding: 20px; margin-bottom: 20px; }
        .stl-viewer { width: 100%; height: 400px; }
        .comments { margin-top: 20px; }
        .comment { background: #f0f0f0; padding: 10px; margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>3D Model Testing Platform</h1>

    <div id="models">
        <!-- Model containers will be dynamically added here -->
    </div>

    <template id="model-template">
        <div class="model-container">
            <h2 class="model-name"></h2>
            <div class="stl-viewer"></div>
            <p class="model-description"></p>
            <a href="#" class="download-link">Download STL</a>
            <button class="test-button">Mark as Tested</button>
            <div class="comments">
                <h3>Comments</h3>
                <div class="comment-list"></div>
                <textarea class="comment-input" placeholder="Add a comment"></textarea>
                <button class="add-comment">Add Comment</button>
            </div>
        </div>
    </template>

    <script>
        const models = [
            { name: "1x1x2 Rubik's cube", src: "model/1x1x2_v1.stl", description: "1x1x2 Rubik's cube", downloadUrl: "model/1x1x2_v1.stl" },
            { name: "Model 2", src: "path/to/model2.stl", description: "Description for Model 2", downloadUrl: "path/to/model2.stl" }
        ];

        function loadSTL(containerElement, stlPath) {
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, containerElement.clientWidth / containerElement.clientHeight, 0.1, 1000);
            const renderer = new THREE.WebGLRenderer();
            renderer.setSize(containerElement.clientWidth, containerElement.clientHeight);
            containerElement.appendChild(renderer.domElement);

            const controls = new THREE.OrbitControls(camera, renderer.domElement);

            const loader = new THREE.STLLoader();
            loader.load(stlPath, function (geometry) {
                const material = new THREE.MeshPhongMaterial({ color: 0xAAAAAA, specular: 0x111111, shininess: 200 });
                const mesh = new THREE.Mesh(geometry, material);
                scene.add(mesh);

                const box = new THREE.Box3().setFromObject(mesh);
                const center = box.getCenter(new THREE.Vector3());
                const size = box.getSize(new THREE.Vector3());

                const maxDim = Math.max(size.x, size.y, size.z);
                camera.position.set(center.x, center.y, center.z + maxDim * 2);
                camera.lookAt(center);

                const light = new THREE.DirectionalLight(0xffffff);
                light.position.set(0, 0, 1);
                scene.add(light);
            });

            function animate() {
                requestAnimationFrame(animate);
                controls.update();
                renderer.render(scene, camera);
            }
            animate();
        }

        const modelsContainer = document.getElementById('models');
        const template = document.getElementById('model-template');

        models.forEach(model => {
            const modelElement = template.content.cloneNode(true);
            modelElement.querySelector('.model-name').textContent = model.name;
            const viewerContainer = modelElement.querySelector('.stl-viewer');
            loadSTL(viewerContainer, model.src);
            modelElement.querySelector('.model-description').textContent = model.description;
            modelElement.querySelector('.download-link').href = model.downloadUrl;

            const testButton = modelElement.querySelector('.test-button');
            testButton.addEventListener('click', () => {
                testButton.textContent = 'Tested';
                testButton.disabled = true;
            });

            const commentList = modelElement.querySelector('.comment-list');
            const commentInput = modelElement.querySelector('.comment-input');
            const addCommentButton = modelElement.querySelector('.add-comment');

            addCommentButton.addEventListener('click', () => {
                const commentText = commentInput.value.trim();
                if (commentText) {
                    const commentElement = document.createElement('div');
                    commentElement.classList.add('comment');
                    commentElement.textContent = commentText;
                    commentList.appendChild(commentElement);
                    commentInput.value = '';
                }
            });

            modelsContainer.appendChild(modelElement);
        });
    </script>
</body>
</html>
