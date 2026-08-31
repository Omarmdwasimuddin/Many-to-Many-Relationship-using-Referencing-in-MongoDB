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
  title!: string;

  @Prop({ type: [{ type: Types.ObjectId, ref: 'Developer' }] })
  developers!: Types.ObjectId;

}

export const ProjectSchema = SchemaFactory.createForClass(Project);
```
---


#### `project.module.ts`
```bash
import { Module } from '@nestjs/common';
import { ProjectService } from './project.service';
import { ProjectController } from './project.controller';
import { MongooseModule } from '@nestjs/mongoose';
import { Developer, DeveloperSchema } from './schemas/developer.schema';
import { Project, ProjectSchema } from './schemas/project.schema';

@Module({
  imports: [
    MongooseModule.forFeature([
      { name: Developer.name, schema: DeveloperSchema },
      { name: Project.name, schema: ProjectSchema }
    ])
  ],
  providers: [ProjectService],
  controllers: [ProjectController]
})
export class ProjectModule {}
```

#

#### `project.service.ts`
```bash
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Developer } from './schemas/developer.schema';
import { Model } from 'mongoose';
import { Project } from './schemas/project.schema';

@Injectable()
export class ProjectService {
    constructor(
        @InjectModel(Developer.name) private developerModel: Model<Developer>,
        @InjectModel(Project.name) private projectModel: Model<Project>,
    ) {}

    async seed(): Promise<{ dev1: Developer; dev2: Developer }> {
        const [projectA, projectB] = await Promise.all([
            this.projectModel.create({ title:  'NestJS CRM'}),
            this.projectModel.create({ title:  'MongoDB Analytics'}),
        ]);

        const [dev1, dev2] = await Promise.all([
            this.developerModel.create({ name: 'Alice', projects: projectA._id }),
            this.developerModel.create({ name: 'Bob', projects: projectB._id }),
        ]);

        await Promise.all([
            this.projectModel.findByIdAndUpdate(projectA._id, { $set: { developers: [dev1._id] } }),
            this.projectModel.findByIdAndUpdate(projectB._id, { $set: { developers: [dev2._id] } }),
        ])
        return { dev1, dev2 };
    }

    async getDevelopers(): Promise<Developer[]> {
        return this.developerModel.find().populate('projects').lean();
    } 

    async getProjects(): Promise<Project[]> {
        return this.projectModel.find().populate('developers').lean();
    }

}
```

#

#### `project.controller.ts`
```bash
import { Controller, Get, Post } from '@nestjs/common';
import { ProjectService } from './project.service';

@Controller('project')
export class ProjectController {
    constructor(private readonly projectService: ProjectService) {}

    @Post('seed')
    async seedData() {
        return this.projectService.seed();
    }

    @Get('developers')
    async getDevelopers() {
        return this.projectService.getDevelopers();
    }

    @Get('projects')
    async getProjects() {
        return this.projectService.getProjects();
    }
}
```
---


>## OUTPUT
>
><img width="703" height="653" alt="image" src="https://github.com/user-attachments/assets/19b03d34-495c-4cc3-856d-ba4383627d98" />
>
>#
>
><img width="720" height="697" alt="image" src="https://github.com/user-attachments/assets/03d47887-7f4c-47f8-8a8d-b24060532f6f" />
>
>#
>
><img width="708" height="711" alt="image" src="https://github.com/user-attachments/assets/68722dd5-4f81-4092-80d7-acfe99899ff1" />
---
