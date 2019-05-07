# MongoDB

**MongoDB almost NEVER report syntax error / null pointer: all error is LOGICAL**

## Admin

### Server

To make sure that your MongoDB instance is running on Windows, run the following command from the Windows command prompt:

    tasklist /FI "IMAGENAME eq mongod.exe"

remote login to db

    C:\Users\cs>mongo [your_ip]:27017
    > use iHealth
    switched to db iHealth
    > db.auth('admin','admin123')
    1
    > db.getCollectionNames()
    [ ]


## Basics

    db.inventory.insertMany([
       // MongoDB adds the _id field with an ObjectId if _id is not present
       { item: "notebook", qty: 50, status: "A",
           size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
       { item: "paper", qty: 100, status: "D",
           size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
       { item: "planner", qty: 75, status: "D",
           size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
       { item: "postcard", qty: 45, status: "A",
           size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] },
        {
            item: "journal", 
            qty: 25, 
            status: "A",
            size: { 
                h: 14, 
                w: 21, 
                uom: "cm" 
            }, 
            tags: ["blank", "red" ]
        },
    ]);

    db.books.insertMany( [
       { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
       { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
       { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
       { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
       { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
    ]);


    db.inventory.find();

    db.inventory.find({
        status: "D"
    });

    db.inventory.find({
        status: {
            $in: ["A", "D"],
        },
    })

**Projection**

    // _id: 0 will hide id.
    db.inventory.find(
        {
            status: {
                $in: ["A", "D"],
            },
        },
        {
            "tags": 1,
            _id: 0,
        }
    )

    db.inventory.find( { status: { $in: ["A", "D"], }, }, {_id: 1} );

**Inequality Query**

**lt: less than; gte: greater than or equal to**

    db.inventory.find({
        status: "A",
        qty: {
            $lt: 30
        },
    })

    db.inventory.find({
        $or: [
            {status: "A"},
            {qty: {$lt: 30}},
        ]
    })


$set and $currentDate

    db.inventory.updateOne(
        { item: "paper" },
        {
            $set: {
                "size.uom": "cm",
                status: "P",
            },
            $currentDate: {
                lastModified: true,
            }
        }
    );

[link](https://docs.mongodb.com/manual/tutorial/update-documents/#update-a-single-document)


    db.inventory.find({
        $or: [
            {
                "size.uom": "in",
                "size.h": { $gte: 8.75, },
            },
            {
                "size.uom": "cm",
                "size.h": { $gte: 22.225, },
            },
        ]
    });

    db.inventory.updateMany(
        {$or: [
            {
                "size.uom": "in",
                "size.h": { $gte: 8.75, },
            },
            {
                "size.uom": "cm",
                "size.h": { $gte: 22.225, },
            },
        ]},
        {
            $set: { status: "B", },
        }
    );

    db.inventory.deleteMany({ "size.uom" : "in" });


**join search**

    original_id = ObjectId()
    db.places.insert({
        "_id": original_id,
        "name": "Broadway Center",
        "url": "bc.example.net"
    })

    db.people.insert({
        "name": "Erin",
        "places_id": original_id,
        "url":  "bc.example.net/Erin"
    })

    db.place.find(db.people.find({name: "Erin"}, {_id: 1}));




// Part 2


    db.restaurant.insertMany([
    {
        Name: "Morris Park Bake Shop", 
        Number: "1007", 
        Street: "Morris Park Ave", 
        ZIP_Code: "10462", 
        Borough: "Bronx ", 
        Cuisine: "Bakery ", 
        Inspections: [
            { Inspection_Date: "1393804800000", Grade: "A",  },
            { Inspection_Date: "1358985600000", Grade: "B",  },
        ]
    },

    {
        Name: "Wendys ", 
        Number: "1602", 
        Street: "Shore Parkway", 
        ZIP_Code: "11214", 
        Borough: "Brooklyn ", 
        Cuisine: "Hamburgers ", 
        Inspections: [
            { Inspection_Date: "1421193600000", Grade: "A",  },
            { Inspection_Date: "1409184000000", Grade: "A",  },
        ]
    },

    {
        Name: "Wendys ", 
        Number: "1661", 
        Street: "Hylan Blvd", 
        ZIP_Code: "10305", 
        Borough: "Staten Island", 
        Cuisine: "Hamburgers ", 
        Inspections: [
            { Inspection_Date: "1417651200000", Grade: "A",  },
            { Inspection_Date: "1334793600000", Grade: "B",  },
        ]
    },

]);


    db.restaurant.find(
    {
        "Inspections.Grade": "B",
    },
    {
        _id: 0,
        Name: 1,
        Number: 1,
    }
    );

    // working area
    [
        { Inspection Date: "1393804800000", Grade: "A",  },
        { Inspection Date: "1358985600000", Grade: "B",  },
    ]
    [
        { Inspection Date: "1421193600000", Grade: "A",  },
        { Inspection Date: "1409184000000", Grade: "A",  },
    ]
    [
        { Inspection Date: "1417651200000", Grade: "A",  },
        { Inspection Date: "1334793600000", Grade: "B",  },
    ]

## Homework 2

    db.students.insertMany([
        {  "Student ID": 1,  "First Name": "Dave",  "Last Name": "Davis", "courses": [
            {  "Course ID": 13,  "Course Name": "Biology",  "Grade": 78 },
            {  "Course ID": 11,  "Course Name": "Calculus",  "Grade": 76 }
        ]},
        {  "Student ID": 2,  "First Name": "John",  "Last Name": "Johnson", "courses": [
            {  "Course ID": 10,  "Course Name": "American History",  "Grade": 90 },
            {  "Course ID": 12,  "Course Name": "Physics",  "Grade": 95 }
        ]},
        {  "Student ID": 3,  "First Name": "Thomas",  "Last Name": "Thompson", "courses": [
            {  "Course ID": 10,  "Course Name": "American History",  "Grade": 88 },
            {  "Course ID": 12,  "Course Name": "Physics",  "Grade": 79 },
            {  "Course ID": 13,  "Course Name": "Biology",  "Grade": 80 }  
        ]}
    ]);

### Select by element in list, sort by value inside an list

Query the first and last name of the student that scored the highest in Biology.

**select element in list**

**include space in queey field**


    db.students.find({"courses.Course Name": "Biology"});

    db.students.find({"courses.Course Name": "Biology"}, 
        {
            "First Name": 1, "Last Name": 1, "_id": 0
        }
    );

    db.students.find({"courses.Course Name": "Biology"}, 
        {
            "First Name": 1, "Last Name": 1, "_id": 0,
            "courses": { $elemMatch: {"Course Name": "Biology"} }
        }
    ).sort({"courses.Grade": -1}).limit(1);


## Draft

    db.test0424.insertMany( [
       { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
       { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
       { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
       { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
       { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
    ]);

    db.test0424.find({"instock.warehouse": "A"});

    db.test0424.find({"instock.warehouse": "A"}, 
        {"instock": { $elemMatch: { warehouse: "A"}}}
    );
    db.test0424.find(
        {"instock.warehouse": "A"}, 
        { _id: 0, item: 1,
            "instock": { $elemMatch: { warehouse: "A"}}
        }
    ).sort(
        {"instock.qty": -1}
    ).limit(1);




