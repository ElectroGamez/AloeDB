<p align="center">
	<img src="https://raw.githubusercontent.com/Kirlovon/AloeDB/master/other/head.png" alt="AloeDB Logo" width="256">
</p>

<h3 align="center">AloeDB-Node</h3>
<p align="center"><i>Light local storage database for NodeJS</i></p>

<p align="center">
    <b>Work in progress of a Work in progress!</b><br>
	<span>Forked from <a href="https://github.com/Kirlovon/AloeDB">Kirlovon their Deno package!</a></span>
</p>

<br>

## Port

This is a port of the Deno package: https://github.com/Kirlovon/AloeDB

## Features

-   ✨ Simple to use API, similar to [MongoDB](https://www.mongodb.com/)!
-   🚀 Optimized for a large number of operations.
-   ⚖ No dependencies!
-   📁 Stores data in readable JSON file.

<br>

## Importing
```typescript
import { Database } from "aloedb-node";
```

## Examples
### Using an interface
```typescript
import { Database } from "aloedb-node";

// Structure of stored documents
interface Film {
	title: string;
	year: number;
	film: boolean;
	genres: string[];
	authors: { director: string };
}

(async () => {
	// Initialization
	const db = new Database<Film>("./path/to/the/file.json");

	// Insert operations
	await db.insertOne({
		title: "Drive",
		year: 2012,
		film: true,
		genres: ["crime", "drama", "noir"],
		authors: { director: "Nicolas Winding Refn" },
	});

	// Search operations
	const found = await db.findOne({ title: "Drive", film: true });
	console.log(found);

	// Update operations
	await db.updateOne({ title: "Drive" }, { year: 2011 });

	// Delete operations
	await db.deleteOne({ title: "Drive" });
})();
```

### Class without a separate interface (Also using uuids and timestamps)
```typescript
import { Database } from "aloedb-node";
import { v1 as uuidv1 } from "uuid";

const db = new Database<Omit<Weather, "save" | "delete" | "update">>("./db/weather.json");

export class Weather {
    id: string;
    timestamp: number;
    temperature: number;
    humidity: number;

    constructor(data: Omit<Weather, "save" | "delete" | "update" | "id" | "timestamp"> & Partial<Pick<Weather, "id" | "timestamp">>) {
        this.id = data.id ?? uuidv1();
        this.timestamp = data.timestamp ?? new Date().getTime();
        this.temperature = data.temperature;
        this.humidity = data.humidity;
    }

    save() {
        return db.insertOne(this);
    }

    delete() {
        return db.deleteOne({ id: this.id });
    }

    update() {
        return db.updateOne({ id: this.id }, this);
    }

    static async findOne(query: Partial<Omit<Weather, "save" | "delete" | "update">>): Promise<Weather | null> {
        const object = await db.findOne(query);
        if (object) return new Weather(object);
        return null;
    }

    static async findMany(query: Partial<Omit<Weather, "save" | "delete" | "update">>): Promise<Weather[]> {
        const objects = await db.findMany(query);

        return objects.map((obj) => {
            return new Weather(obj);
        });
    }
}
```

### Class with seperate Interface
```typescript
import { Database } from "aloedb-node";
import { v1 as uuidv1 } from "uuid";

const db = new Database<IUser>("./db/user.json");

interface IUser {
    id?: string;
    email: string;
    hash: string;
}

export class User {
    id: string;
    email: string;
    hash: string;

    constructor(data: IUser) {
        this.id = data.id ?? uuidv1();
        this.email = data.email;
        this.hash = data.hash;
    }

    save() {
        return db.insertOne(this);
    }

    delete() {
        return db.deleteOne({ id: this.id });
    }

    update() {
        return db.updateOne({ id: this.id }, this);
    }

    static async findOne(query: Partial<IUser>): Promise<User | null> {
        const object = await db.findOne(query);
        if (object) return new User(object);
        return null;
    }

    static async findMany(query: Partial<IUser>): Promise<User[]> {
        const objects = await db.findMany(query);

        return objects.map((obj) => {
            return new User(obj);
        });
    }
}
```

### Using one of these classes
```typescript
// Create a new entity
const newWeather = new Weather({temperature: 15, humidity: 32});
await newWeather.save();

// Update a existing entity
newWeather.temperature = 16;
newWeather.update();

// Find and existing entity
const oldWeather = await Weather.findOne({ id: "13d2e1c2-feda-498f-8540-dd92f1087161" });

// Delete a existing entity
await oldWeather.delete();

// Get an array of many entities
const moreWeathers = await Weather.findMany({});

for (const weather of moreWeathers) {
	console.log(weather.id);
}
```

## Support
If you want to buy someone a coffee, try to get in contact with the <a href="https://github.com/Kirlovon/AloeDB">contributors from this package</a>, they did 99.9% of the work, I just made it run in node because I was frustrated with existing packages.