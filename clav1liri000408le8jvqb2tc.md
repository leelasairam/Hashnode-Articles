---
title: "Integrating Salesforce with NodeJs using JsForce"
seoTitle: "Integrating Salesforce with NodeJs using JsForce"
seoDescription: "Integrating Salesforce with NodeJs using JsForce"
datePublished: Thu Nov 24 2022 12:20:04 GMT+0000 (Coordinated Universal Time)
cuid: clav1liri000408le8jvqb2tc
slug: integrating-salesforce-with-nodejs-using-jsforce
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/npxXWgQ33ZQ/upload/v1669292297600/OPq3JLnBt.jpeg
tags: express, nodejs, reactjs, salesforce, jsforce

---


![Demo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r54ije2ue2ryxa9s6bnh.png)

JsForce is the best library to integrate the salesforce with Nodejs. In this blog we see how to use JsForce library (CRUD) in node and use react as frontend library.

## Prerequests ##

- Salesforce Org
- NodeJs Installed _(Download NodeJs from [here](https://nodejs.org/en/download/).)_

## Packages Required ##
<u>Backend</u>

1. Express - `npm install express`
2. Nodemon - `npm install nodemon`
3. Cors - `npm install cors`
4. JsForce - `npm install jsforce`

<u>Frontend</u>

1. Axios - `npm install axios`
2. Bootstrap for react - `npm install react-bootstrap bootstrap`

## Github Repository ##
[react-node-salesforce](https://github.com/leelasairam/react-node-salesforce)

## Connecting Salesforce org with JsForce ##

Create seperate folder "Routes" to keep `index.js` clean. For example, keep all `/salesforce` in `Routes > salesforce.js` and include it in `index.js` as below.

```
app.use('/salesforce',salesforceRouter);
```
Now, whenever REST call comes with /salesforce, `index.js` searches in `Routes > salesforce.js`
```
function connection(req, res, next){
    const username = 'username';
      const password = 'password + security token';
      conn.login(username, password, function(err, userInfo) {
        if (err) { return console.error(err); }
        console.log(conn.accessToken);
        console.log(conn.instanceUrl);
        console.log("User ID: " + userInfo.id);
        console.log("Org ID: " + userInfo.organizationId);
      });
      next()
} 
```
To reset security token, go to salesforce org > Click on profile icon > Select 'Reset Token'

## Query Records ##

To search the account, pass the query in call after `q` like this `http://localhost:8000/salesforce/accounts?key=${term}`

```
var records = [];
conn.query("SELECT Id, Name FROM Account", function(err, result) {
  if (err) { return console.error(err); }
  console.log("total : " + result.totalSize);
  console.log("fetched : " + result.records.length);
});
```

## Create Record ##


![Create Account](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hijue3v2kzm7dwaunko4.png)

To get data from request use `req.body`, it will get the data that passed from frontend using axios
```
router.post("/new-account", async(req,res) => {
    const acc1 = req.body;
    await conn.sobject("Account").create({ Name : acc1.name, Rating : acc1.rating, Phone : acc1.phone}, function(err, ret) {
        if (err || !ret.success) {
             return console.error(err, ret); 
        }
        console.log("Created record id : " + ret.id);
        res.send({msg : `Created record id : ${ret.id}`})
      });
});
```

## Update Record ##

Updating is similar to creating the record but we have to pass record Id too. 

```
router.patch("/edit-account", async(req,res) => {
    await conn.sobject("Account").update({ 
        Id : req.body.id,
        Name : req.body.name,
        Phone : req.body.phone,
        Rating : req.body.rating,
      }, function(err, ret) {
        if (err || !ret.success) {
             return console.error(err, ret); 
        }
        console.log('Updated Successfully : ' + ret.id);
        res.send({msg : `Updated record id : ${ret.id}`})
      });
})
```

## Delete Record ##

For deleting record, we just need to pass the Id of the record. We pass the Id in url itself `(http://localhost:8000/salesforce/delete-account/${id})`.

<u>React </u>

```
const DeleteAccount = async(id) => {
    const res = await axios.delete(`http://localhost:8000/salesforce/delete-account/${id}`);
    const result = await res.data;
    console.log(result.msg);
    alert(result.msg);
  }
```
<u>Node</u>

```
router.delete("/delete-account/:id", async(req,res) => {
      conn.sobject("Account").destroy(req.params.id, function(err, ret) {
        if (err || !ret.success) {
             console.error(err, ret); 
             res.send({msg : err + ret})
        }
        else{
          console.log('Deleted Successfully : ' + ret.id);
         res.send({msg : `Deleted record id : ${ret.id}`})
        }
      });
})
```

## Frontend Part ##

Used React in frontend part. To make calls, used Axios. To get more info about Axios, click [here](https://axios-http.com/docs/example). Used `Bootstrap` along with css for styling.

```
import React, { useState, useEffect } from "react";
import "./App.css";
import axios from "axios";
import 'bootstrap/dist/css/bootstrap.min.css';
import Button from 'react-bootstrap/Button';
import Modal from 'react-bootstrap/Modal';

function App() {
  const [message, setMessage] = useState({});
  const [accounts, setAccounts] = useState([]);
  const [accname, setAccname] = useState("");
  const [show, setShow] = useState(false);
  const [rating, setRating] = useState("");
  const [phone, setPhone] = useState("");
  const [id,setId] = useState("");
  const [showedit,setShowedit] = useState(false);
  const [term, setTerm] = useState("");

  const handleClose = () => setShow(false);
  const handleShow = () => {
    setId("");
    setAccname("");
    setPhone("");
    setRating("");
    setShowedit(false);
    setShow(true);
  }

  useEffect(() => {
    GetData();
    GetAccounts();
   }, []);

  const GetData = async()=>{
    const res = await axios.get("http://localhost:8000/salesforce/home");
    const result = await res.data;
    setMessage(result);
  }

  const GetAccounts = async() => {
    const res = await axios.get(`http://localhost:8000/salesforce/accounts?key=${term}`);
    //console.log(res); 
    const result = await res.data;
    console.log(result); 
    setAccounts(result.acc);
  }

  const CreateAccount = async() => {
    if(accname && phone && rating){
      const res = await axios.post('http://localhost:8000/salesforce/new-account', {
      name: accname,
      phone: phone,
      rating: rating
    });
    const result = await res.data;
    console.log(result.msg);
    setShow(false)
    }
    else{
      alert("Fill all fields.")
    }
  }

  const EditAccount = async(id,name,phone,rating) => {
    setId(id);
    setAccname(name);
    setPhone(phone);
    setRating(rating);
    setShowedit(true);
    setShow(true);
  }

  const SaveEdit = async() => {
    const res = await axios.patch('http://localhost:8000/salesforce/edit-account', {
      name: accname,
      id:id,
      phone:phone,
      rating:rating
    });
    const result = await res.data;
    console.log(result.msg);
    setShowedit(false);
    setShow(false);
  }

  const DeleteAccount = async(id) => {
    const res = await axios.delete(`http://localhost:8000/salesforce/delete-account/${id}`);
    const result = await res.data;
    console.log(result.msg);
    alert(result.msg);
  }


  return (
    <>
    <div style={{paddingLeft:"5rem",paddingRight:"5rem"}}>
      <div style={{marginBottom:"1rem",textAlign:"center"}}>
      <p style={{fontWeight:"600"}}>Connected (<a href={message.url}>{message.url}</a>)</p>
        <div>
          <input onChange={(event) => setTerm(event.target.value)} placeholder="Search..."/>
          <button onClick={GetAccounts} style={{marginLeft:"0.2rem",backgroundColor:"#D2DAFF",borderColor:"#D2DAFF"}}>SEARCH</button>
          <button style={{marginLeft:"0.8rem",backgroundColor:"#CFF5E7",borderColor:"#CFF5E7"}} onClick={handleShow}>+ CREATE NEW ACCOUNT</button> 
        </div>
      </div>
      {accounts.map((i,j) => {
        return(
          <div key={i.Id} style={{padding:"0.5rem",backgroundColor:"#2C3639",color:"white",marginBottom:"0.3rem"}}>
            <p style={{fontWeight:"600"}}><span style={{color:"#FFEFD6",fontWeight:"900",textDecoration:"underline"}}>{i.Name}</span>&nbsp;({i.Id})</p>
            <p style={{fontWeight:"600"}}>{(i.Phone) ? "Phone : "+i.Phone : "NA"} &nbsp; {(i.Rating) ? "Rating : "+i.Rating : "NA"}</p>
            
            <div>
              <button style={{color:"purple",border:"none"}} onClick={() => EditAccount(i.Id,i.Name,i.Phone,i.Rating)}>EDIT</button>
              <button style={{marginLeft:"0.4rem",color:"#CF0A0A",border:"none"}} onClick={() => DeleteAccount(i.Id)}>DELETE</button>
            </div>
          </div>
        )
      })}
    </div>

    <Modal show={show} onHide={handleClose}>
        <Modal.Header closeButton>
          <Modal.Title>{showedit === false ? "New Account" : "Edit Account"}</Modal.Title>
        </Modal.Header>
        <Modal.Body>
        <label for="name">Account name :</label><br/>
         <input value={accname} name="name" style={{}} onChange={(event) => setAccname(event.target.value)} placeholder="Jack"/><br/><br/>
         <label for="phone">Phone :</label><br/>
         <input value={phone} name="phone" style={{}} onChange={(event) => setPhone(event.target.value)} placeholder="Ex : 9867453421"/><br/><br/>
         <label for="rating">Rating :</label><br/>
         <select value={rating} name="rating" onChange={(event) => setRating(event.target.value)} style={{width:"190px"}}>
         <option value=""></option>
          <option value="Hot">Hot</option>
          <option value="Warm">Warm</option>
          <option value="Cold">Cold</option>
         </select>
        </Modal.Body>
        <Modal.Footer>
          <Button variant="secondary" onClick={handleClose}>
            Close
          </Button>
          {showedit === false ? <Button variant="primary" onClick={CreateAccount}>
            Submit
          </Button> : <Button variant="primary" onClick={SaveEdit}>
            Save Changes
          </Button> }
        </Modal.Footer>
      </Modal>
    

    </>
  );
}

export default App
```

---
Thanks for reading.

