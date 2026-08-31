## Many-to-Many Relationship using Referencing in MongoDB


#### Create module, service, controller
```bash
nest g module project
```
```bash
nest g service project
```
```bash
nest g controller project
```
---


>#### create koro file folder- schemas/developer.schema.ts & schemas/project.schema.ts
#### `developer.schema.ts`
```bash
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { Document, Types } from "mongoose";


@Schema({ timestamps: true })
export class Developer extends Document {

  @Prop({ required: true })
  name!: string;

  @Prop({ type: [{ type: Types.ObjectId, ref: 'Project' }] })
  projects!: Types.ObjectId;

}

export const DeveloperSchema = SchemaFactory.createForClass(Developer);
```

#

#### `project.schema.ts`
```bash
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { Document, Types } from "mongoose";


@Schema({ timestamps: true })
export class Project extends Document {

  @Prop({ required: true })
  title: string;

  @Prop({ type: [{ type: Types.ObjectId, ref: 'Developer' }] })
  developers: Types.ObjectId;

}

export const ProjectSchema = SchemaFactory.createForClass(Project);
```
---
