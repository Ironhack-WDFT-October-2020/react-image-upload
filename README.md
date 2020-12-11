# React Image Upload

### Configuration of cloudinary in the backend did not change

### We now have a route in the backend that uploads the file and returns the data from cloudinary as json to the frontend

```js
// routes/file-upload.js
router.post('/', uploader.single("imageURL"), (req, res, next) => {
  console.log('file is: ', req.file)
  console.log('public id: ', req.file.filename);
  if (!req.file) {
    next(new Error('No file uploaded!'));
    return;
  }

  res.json({ secure_url: req.file.path, public_id: req.file.filename });
})
```

### In the frontend we have now a service that posts the image to this route from above

```js
// client/src/services/upload.js
import axios from 'axios';

const errorHandler = err => {
  // console.error(err);
  throw err;
};

const Axios = axios.create({
  baseURL: 'http://localhost:3000',
  /* other custom settings */
});

export default {
  handleUpload(file) {
    console.log('file to be handled: ', file);
    return axios.post('api/upload', file)
      .then(res => res.data)
      .catch(errorHandler);
  }
}
```

### In ProjectForm we first post the image using the service and then we post the project with the data from cloudinary to the projects create route.

### To post the file with JavaScript we use the FormData Object.
```js
    const uploadData = new FormData();
    uploadData.append("imageURL", e.target.files[0]);
```

```js
// client/src/ProjectForm
  handleFileUpload = e => {
    console.log("The file to be uploaded is: ", e.target.files[0]);
    this.setState({
      imageSelected: true
    });
    const uploadData = new FormData();
    uploadData.append("imageURL", e.target.files[0]);

    service.handleUpload(uploadData)
      .then(response => {
        const imageURL = response.secure_url;
        const publicID = response.public_id;
        console.log('res from handleupload: ', response.secure_url);
        this.setState({ imageURL: imageURL, publicID: publicID });
        console.log('new state: ', this.state.imageURL);
        // check if the form already got submitted and only waits for the image upload
        if (this.state.submitted === true) {
          this.handleSubmit();
        }
      })
      .catch(err => {
        this.setState({
          imageSelected: false
        });
        console.log("Error while uploading the file: ", err);
      });
  }
```

### Then after uploading the image we post the project itself

```js
  handleSubmit = event => {
    if (event) {
      event.preventDefault();
    }
    console.log("SUBMIT");
    // check if the image is already uploaded to the cloud or no image was selected
    if (this.state.imageURL || !this.state.imageSelected) {
      // axios.post('http://localhost:5555/api/projects')
      axios
        .post("/api/projects", {
          title: this.state.title,
          description: this.state.description,
          imageURL: this.state.imageURL,
          imagePublicID: this.state.publicID
        })
        .then(response => {
          this.props.refreshData();
          this.setState({
            title: "",
            description: "",
            imageURL: "",
            publicID: ""
          });
        })
        .catch(err => {
          console.log(err);
        });
    } else {
      // set a flag that the project got submitted
      this.setState({
        submitted: true
      })
    }
  };
```

### This is the complete component

```js
import React, { Component } from "react";
import axios from "axios";
import { Form, Button } from "react-bootstrap";
import service from "../services/upload.js";

class ProjectForm extends Component {

  state = {
    title: "",
    description: "",
    imageURL: "",
    publicID: "",
    submitted: false,
    imageSelected: false,
  };

  handleChange = event => {
    this.setState({
      [event.target.name]: event.target.value
    });
  };

  handleFileUpload = e => {
    console.log("The file to be uploaded is: ", e.target.files[0]);
    this.setState({
      imageSelected: true
    });
    const uploadData = new FormData();
    uploadData.append("imageURL", e.target.files[0]);

    service.handleUpload(uploadData)
      .then(response => {
        const imageURL = response.secure_url;
        const publicID = response.public_id;
        console.log('res from handleupload: ', response.secure_url);
        this.setState({ imageURL: imageURL, publicID: publicID });
        console.log('new state: ', this.state.imageURL);
        // check if the form already got submitted and only waits for the image upload
        if (this.state.submitted === true) {
          this.handleSubmit();
        }
      })
      .catch(err => {
        this.setState({
          imageSelected: false
        });
        console.log("Error while uploading the file: ", err);
      });
  }

  handleSubmit = event => {
    if (event) {
      event.preventDefault();
    }
    console.log("SUBMIT");
    // check if the image is already uploaded to the cloud or no image was selected
    if (this.state.imageURL || !this.state.imageSelected) {
      // axios.post('http://localhost:5555/api/projects')
      axios
        .post("/api/projects", {
          title: this.state.title,
          description: this.state.description,
          imageURL: this.state.imageURL,
          imagePublicID: this.state.publicID
        })
        .then(response => {
          this.props.refreshData();
          this.setState({
            title: "",
            description: "",
            imageURL: "",
            publicID: ""
          });
        })
        .catch(err => {
          console.log(err);
        });
    } else {
      // set a flag that the project got submitted
      this.setState({
        submitted: true
      })
    }
  };

  render() {
    return (
      <Form onSubmit={this.handleSubmit}>
        <Form.Group>
          <Form.Label htmlFor="title">Title: </Form.Label>
          <Form.Control
            type="text"
            name="title"
            id="title"
            onChange={this.handleChange}
            value={this.state.title}
          />
        </Form.Group>

        <Form.Group>
          <Form.Label htmlFor="description">Description: </Form.Label>
          <Form.Control
            type="text"
            name="description"
            id="description"
            onChange={this.handleChange}
            value={this.state.description}
          />
        </Form.Group>

        <Form.Group>
          <Form.Label htmlFor="imageURL">Image: </Form.Label>
          <Form.Control
            type="file"
            name="image"
            id="image"
            onChange={this.handleFileUpload}
          />
        </Form.Group>

        <Button type="submit">Create a project</Button>
      </Form>
    );
  }
}

export default ProjectForm;
```
### The two booleans in the state (submitted and imageSelected) are optional. They were added to prevent a small bug: You could click upload file and then immediately after click submit and then the project would be submitted but without the data from cloudinary. This is to prevent that.