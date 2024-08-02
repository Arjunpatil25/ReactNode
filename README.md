//--------------------------------------------------- dbcoonect.js ----------------------------------------------<br>
const mysql=require("mysql2")

var mysqlconnection = mysql.createConnection({
    host:'localhost',
    user:'root',
    password:'root',
    database:'wpt',
    port:3306
})

mysqlconnection.connect((error)=>{
    if(error){
        console.log(error)
    }
    else{
        console.log("connected")
    }
})

module.exports=mysqlconnection;

//-------------------------------------------------------- app.js --------------------------------------------------------<br>
const express = require("express");
const app = express();
const path = require("path");
const routes=require("./routes/router")
const bodyparser=require("body-parser")

app.set("views",path.join(__dirname,"views"))
app.set("view engine","ejs");

app.use(bodyparser.urlencoded({extended:false}))
app.use("/",routes)
app.listen(3003,function(){
console.log("servor started on 3003")
})

module.exports=app;

//----------------------------------------------------- router.js ---------------------------------------------------<br>
const express = require("express")
const myrouter=express.Router();
const connection=require("../db/dbconnect");

myrouter.get("/courses",function(req,res){
    connection.query(
        "select * from course",function(err,data){
            if(err){
                res.status(500).send("errorrr")

            }else {
                res.status(200).render("display",{coursedata:data})
            }
        }
    )
})
//to show empty form to add data
myrouter.get("/addcourseform",function(req,res){
    res.render("add_course")
})

//add course
myrouter.post("/insertcourse",function(req,res){
    connection.query("insert into course values(?,?,?,?)",[req.body.cid,req.body.cnmae,req.body.fees,req.body.duration],function(err,data){
        if(err){
            res.status(500).send("error")
        }else {
            res.redirect("/courses")
        }
    })

})
//delete 
myrouter.get("/deletecourse/:id",function(req,res){
    connection.query("delete from course where cid=?",[req.params.id],function(err,data){
        if(err){
            res.status(500).send("error")
        }else {
            res.redirect("/courses")}
        })

})
//
myrouter.get("/editcourse/:id",function(req,res){
    connection.query("select * from course where cid=?",[req.params.id],function(err,data){
        if(err){
            res.status(500).send("error")
        }else {
            res.render("update_course",{editdata:data[0]})}
        })
        
})
//update 
myrouter.post("/updatecourse",function(req,res){
    connection.query("update course set cnmae=?,fees=?,duraction=? where cid=?",[req.body.cnmae,req.body.fees,req.body.duraction,req.body.cid],function(err,data){
        if(err){
            res.status(500).send("error")
        }else {
            res.redirect("/courses")
        }
        
})
})
module.exports=myrouter;

//------------------------------------------------add_course.ejs -----------------------------------------------<br>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="/insertcourse" method="post">
        course id:<input type="text" name="cid" id="cid"><br>
        course name:<input type="text" name="cnmae" id="cnmae"><br>
        course fees:<input type="text" name="fees" id="fees"><br>
        course duration:<input type="text" name="duration" id="duration"><br>
        <button type="submit" >Add course </button>
    </form>
</body>
</html>

//------------------------------------------- display ------------------------------------------------------<br>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="/addcourseform">
        <button type="submit" >add new course</button>

    </form>

    <table>
        <tr>
            <th>course id </th>
            <th>name</th>
            <th>fees</th>
            <th>duraction</th>

        </tr>
        <% for(var i=0;i<coursedata.length;i++){%>

       <tr>
        <td><%=coursedata[i].cid%></td>
        <td><%=coursedata[i].cnmae%></td>
        <td><%=coursedata[i].fees%></td>
        <td><%=coursedata[i].duraction%></td>
        <td>
            <a href="deletecourse/<%=coursedata[i].cid%>">deletecourse</a>
            <a href="editcourse/<%=coursedata[i].cid%>">editcourse</a>
        </td>
       </tr>

     <%}%>
    </table>
</body>
</html>

//------------------------------------------- update_course -----------------------------------------------<br>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="/updatecourse" method="post">
        Course ID: <input type="text" name="cid" id="cid" value="<%= editdata.cid %>" readonly><br>
        Course Name: <input type="text" name="cnmae" id="cnmae" value="<%= editdata.cnmae %>"><br>
        Course Fees: <input type="text" name="fees" id="fees" value="<%= editdata.fees %>"><br>
        Course Duration: <input type="text" name="duraction" id="duraction" value="<%= editdata.duraction %>"><br>
        <button type="submit">Update Course</button>
    </form>
    
</body>
</html>
