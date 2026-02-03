# Template Service - מדריך מבנה הפרויקט

## תוכן עניינים

- [סקירה כללית](#סקירה-כללית)
- [טכנולוגיות (Stack)](#טכנולוגיות-stack)
- [מבנה התיקיות](#מבנה-התיקיות)
- [ארכיטקטורה](#ארכיטקטורה)
- [פירוט הקבצים](#פירוט-הקבצים)
  - [קבצי שורש](#קבצי-שורש)
  - [תיקיית Express](#תיקיית-express)
  - [תיקיית Feature (template)](#תיקיית-feature-template)
  - [תיקיית Utils](#תיקיית-utils)
- [איך ליצור Feature חדש](#איך-ליצור-feature-חדש)
- [Environment Variables](#environment-variables)

---

## סקירה כללית

טמפלייט זה מהווה בסיס לשירות REST API מבוסס Node.js עם Express ו-MongoDB.
הפרויקט בנוי בארכיטקטורת שכבות (Layered Architecture) עם דגש על:

- **Type Safety** - שימוש ב-TypeScript עם הגדרות strict
- **Validation** - וולידציה בכניסה לכל endpoint עם Zod
- **Error Handling** - טיפול מרוכז בשגיאות
- **Clean Code** - הפרדה ברורה בין שכבות

---

## טכנולוגיות (Stack)

### Runtime & Language

| טכנולוגיה | גרסה | תיאור |
|-----------|------|-------|
| Node.js | 20.x | סביבת ריצה |
| TypeScript | 5.9.x | שפת תכנות |
| ES Modules | ESNext | מערכת מודולים |

### Core Framework

| טכנולוגיה | גרסה | תיאור |
|-----------|------|-------|
| Express | 5.x | Web Framework |
| Mongoose | 8.x | MongoDB ODM |
| Zod | 4.x | Schema Validation |

### Security & Middleware

| טכנולוגיה | תיאור |
|-----------|-------|
| Helmet | אבטחת HTTP Headers |
| CORS | Cross-Origin Resource Sharing |

### Logging

| טכנולוגיה | תיאור |
|-----------|-------|
| Winston | מערכת לוגים |
| express-winston | Middleware ללוגים |

### Configuration

| טכנולוגיה | תיאור |
|-----------|-------|
| dotenv | טעינת משתני סביבה מקובץ |
| env-var | פרסור ווולידציה של משתני סביבה |

### Development Tools

| טכנולוגיה | תיאור |
|-----------|-------|
| ESLint | בדיקת קוד |
| Prettier | פורמט קוד |
| Vitest | בדיקות יחידה |
| Supertest | בדיקות HTTP |
| mongodb-memory-server | MongoDB בזיכרון לבדיקות |
| tsc-watch | Hot reload בפיתוח |

---

## מבנה התיקיות

```
project-root/
├── src/
│   ├── index.ts                    # Entry point - הפעלת האפליקציה
│   ├── config.ts                   # קונפיגורציה - משתני סביבה
│   ├── corsConfig.ts               # הגדרות CORS
│   │
│   ├── express/
│   │   ├── server.ts               # מחלקת השרת - Express setup
│   │   ├── router.ts               # Router ראשי - חיבור כל ה-routes
│   │   │
│   │   └── [feature]/              # תיקיית Feature (לדוגמה: template)
│   │       ├── interface.ts        # TypeScript interfaces
│   │       ├── model.ts            # Mongoose Schema & Model
│   │       ├── validations.ts      # Zod Schemas לוולידציה
│   │       ├── manager.ts          # Business Logic
│   │       ├── controller.ts       # HTTP Request Handlers
│   │       └── router.ts           # Feature Routes
│   │
│   └── utils/
│       ├── errors.ts               # Custom Error Classes
│       ├── zod.ts                  # Zod Utilities & Types
│       │
│       ├── express/
│       │   ├── error.ts            # Error Middleware
│       │   └── wrappers.ts         # Controller & Validation Wrappers
│       │
│       └── logger/
│           ├── index.ts            # Winston Logger Setup
│           └── middleware.ts       # Express Logger Middleware
│
├── package.json
├── tsconfig.json
├── Dockerfile
├── .eslintrc.cjs
├── .prettierrc.js
└── .env.development / .env.production
```

---

## ארכיטקטורה

הפרויקט בנוי בארכיטקטורת שכבות (Layered Architecture):

```
┌─────────────────────────────────────────────────────────────────┐
│                         Request                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Router Layer                                                    │
│  - הגדרת endpoints                                              │
│  - הפעלת validation middleware                                   │
│  - ניתוב ל-controller המתאים                                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Controller Layer                                                │
│  - קבלת request מטופל ומוולד                                     │
│  - הפעלת הלוגיקה העסקית                                          │
│  - החזרת response                                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Manager Layer (Business Logic)                                  │
│  - לוגיקה עסקית                                                  │
│  - תקשורת עם ה-Model                                             │
│  - טיפול בשגיאות עסקיות                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Model Layer                                                     │
│  - Mongoose Schema                                               │
│  - אינטראקציה עם MongoDB                                         │
└─────────────────────────────────────────────────────────────────┘
```

### זרימת Request

```
Request
   │
   ▼
CORS Middleware ──► Helmet ──► JSON Parser ──► Logger Middleware
   │
   ▼
Router (validateRequest) ──► Controller ──► Manager ──► Model ──► MongoDB
   │
   ▼
Response / Error Middleware
```

---

## פירוט הקבצים

### קבצי שורש

#### `src/index.ts` - Entry Point

נקודת הכניסה של האפליקציה. אחראי על:
- התחברות ל-MongoDB
- הפעלת השרת

```typescript
import mongoose from 'mongoose';
import { config } from './config';
import { Server } from './express/server';
import { logger } from './utils/logger/index';

const { mongo, service } = config;

const initializeMongo = async () => {
    logger.info('Connecting to Mongo...');
    await mongoose.connect(mongo.uri);
    logger.info('Mongo connection established');
};

const main = async () => {
    await initializeMongo();
    const server = new Server(service.port);
    await server.start();
    logger.info(`Server started on port: ${service.port}`);
};

main().catch(logger.error);
```

---

#### `src/config.ts` - Configuration

ניהול קונפיגורציה מרוכז. טוען משתני סביבה ומוודא שהם קיימים.

```typescript
import path from 'path';
import dotenv from 'dotenv';
import env from 'env-var';

const nodeEnv = process.env['NODE_ENV'] ?? 'development';
const envFile = nodeEnv === 'production' ? '.env.production' : '.env.development';
dotenv.config({ path: path.resolve(process.cwd(), envFile) });

export const config = {
    service: {
        port: env.get('PORT').default(8000).required().asPortNumber(),
    },
    mongo: {
        uri: env.get('MONGO_URI').required().asString(),
        templateCollectionName: env.get('TEMPLATE_SERVICE').required().asString(),
    },
    cors: {
        origin: env.get('CORS_ORIGIN').required().asString(),
    },
};
```

**הסבר:**
- `env.get('VAR').required()` - מוודא שהמשתנה קיים
- `asPortNumber()` / `asString()` - מבצע פרסור והמרת טיפוס
- `.default(8000)` - ערך ברירת מחדל אם לא הוגדר

---

#### `src/corsConfig.ts` - CORS Configuration

הגדרות CORS לשליטה על גישה מ-origins אחרים.

```typescript
import { config } from './config';

const { origin } = config.cors;

const corsOptions = {
    origin: origin,
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization'],
};

export default corsOptions;
```

---

### תיקיית Express

#### `src/express/server.ts` - Express Server Class

מחלקת השרת. אחראית על:
- יצירת Express app
- הגדרת middleware chain
- הפעלת השרת

```typescript
import { once } from 'events';
import express from 'express';
import helmet from 'helmet';
import http from 'http';
import cors from 'cors';
import corsOptions from '../corsConfig';

import { errorMiddleware } from '../utils/express/error';
import { loggerMiddleware } from '../utils/logger/middleware';
import { appRouter } from './router';

export class Server {
    private app: express.Application;
    private http?: http.Server;

    constructor(private port: number) {
        this.app = Server.createExpressApp();
    }

    static createExpressApp() {
        const app = express();

        // Middleware Chain - סדר חשוב!
        app.use(cors(corsOptions));      // 1. CORS
        app.use(helmet());               // 2. Security Headers
        app.use(express.json());         // 3. JSON Parser
        app.use(express.urlencoded({ extended: true })); // 4. URL Encoded
        app.use(loggerMiddleware);       // 5. Logger
        app.use(appRouter);              // 6. Routes
        app.use(errorMiddleware);        // 7. Error Handler (תמיד אחרון!)

        return app;
    }

    async start() {
        this.http = this.app.listen(this.port);
        await once(this.http, 'listening');
    }
}
```

---

#### `src/express/router.ts` - Main Application Router

ה-Router הראשי שמחבר את כל ה-feature routers.

```typescript
import { Router } from 'express';
import { templateRouter } from './template/router';

export const appRouter = Router();

// Feature Routes
appRouter.use('/api/template', templateRouter);

// Health Check Endpoints
appRouter.use(['/isAlive', '/isalive', '/health'], (_req, res) => {
    res.status(200).send('alive');
});

// 404 Handler
appRouter.use((_req, res) => {
    res.status(404).send('Invalid Route');
});
```

**הוספת Feature חדש:**
```typescript
import { userRouter } from './user/router';
appRouter.use('/api/user', userRouter);
```

---

### תיקיית Feature (template)

כל Feature בפרויקט מכיל את הקבצים הבאים:

#### `interface.ts` - TypeScript Interfaces

הגדרת הטיפוסים של ה-Feature.

```typescript
// ה-Interface הבסיסי - השדות של המסמך
export interface Template {
    name: string;
    email: string;
}

// ה-Interface של המסמך עם _id מ-MongoDB
export interface TemplateDocument extends Template {
    _id: string;
}
```

**דוגמה ל-Feature אחר (User):**
```typescript
export interface User {
    firstName: string;
    lastName: string;
    email: string;
    age: number;
    isActive: boolean;
}

export interface UserDocument extends User {
    _id: string;
}
```

---

#### `model.ts` - Mongoose Schema & Model

הגדרת ה-Schema וה-Model של MongoDB.

```typescript
import mongoose from 'mongoose';
import { config } from '../../config';
import { TemplateDocument } from './interface';

const TemplateSchema = new mongoose.Schema<TemplateDocument>(
    {
        name: {
            type: String,
            required: true,
        },
        email: {
            type: String,
        },
    },
    {
        versionKey: false,  // מבטל את __v
    },
);

export const TemplateModel = mongoose.model<TemplateDocument>(
    config.mongo.templateCollectionName,
    TemplateSchema
);
```

**דוגמה מורחבת עם אינדקסים:**
```typescript
const UserSchema = new mongoose.Schema<UserDocument>(
    {
        firstName: { type: String, required: true },
        lastName: { type: String, required: true },
        email: { type: String, required: true, unique: true },
        age: { type: Number, min: 0 },
        isActive: { type: Boolean, default: true },
    },
    {
        versionKey: false,
        timestamps: true,  // מוסיף createdAt, updatedAt
    },
);

// הוספת אינדקסים
UserSchema.index({ email: 1 });
UserSchema.index({ firstName: 1, lastName: 1 });
```

---

#### `validations.ts` - Zod Validation Schemas

סכמות וולידציה לכל endpoint. מגדירות מה צריך להיות ב-body, query, params.

```typescript
import { z } from 'zod';
import { zodMongoObjectId } from '../../utils/zod';

// שדות חובה ליצירה
const requiredFields = z
    .object({
        name: z.string(),
        email: z.email(),
    })
    .required();

// GET /api/template - קבלת רשימה עם pagination
export const getByQueryRequestSchema = z.object({
    body: z.object({}),
    query: z
        .object({
            step: z.coerce.number().min(0).default(0),
            limit: z.coerce.number().optional(),
        })
        .extend(requiredFields.partial().shape),  // שדות לחיפוש (אופציונליים)
    params: z.object({}),
});

// GET /api/template/count - ספירה
export const getCountRequestSchema = z.object({
    body: z.object({}),
    query: requiredFields.partial(),  // שדות לחיפוש
    params: z.object({}),
});

// GET /api/template/:id - קבלה לפי ID
export const getByIdRequestSchema = z.object({
    body: z.object({}),
    params: z.object({
        id: zodMongoObjectId,  // וולידציה של ObjectId
    }),
    query: z.object({}),
});

// POST /api/template - יצירה
export const createOneRequestSchema = z.object({
    body: requiredFields,  // כל השדות חובה
    params: z.object({}),
    query: z.object({}),
});

// PUT /api/template/:id - עדכון
export const updateOneRequestSchema = z.object({
    body: requiredFields.partial(),  // שדות אופציונליים
    params: z.object({
        id: zodMongoObjectId,
    }),
    query: z.object({}),
});

// DELETE /api/template/:id - מחיקה
export const deleteOneRequestSchema = z.object({
    body: z.object({}),
    params: z.object({
        id: zodMongoObjectId,
    }),
    query: z.object({}),
});
```

**טיפים לוולידציה:**
```typescript
// טיפוסים נפוצים
z.string().min(1).max(100)        // מחרוזת עם אורך
z.number().int().positive()        // מספר שלם חיובי
z.email()                          // אימייל תקין
z.coerce.number()                  // המרה ממחרוזת למספר (מתאים ל-query params)
z.boolean().default(false)         // בוליאני עם ברירת מחדל
z.array(z.string())                // מערך של מחרוזות
z.enum(['admin', 'user'])          // ערך מתוך רשימה
z.date()                           // תאריך
```

---

#### `manager.ts` - Business Logic Layer

שכבת הלוגיקה העסקית. מבצעת פעולות על המודל.

```typescript
import { DocumentNotFoundError } from '../../utils/errors';
import { Template, TemplateDocument } from './interface';
import { TemplateModel } from './model';

export class TemplateManager {
    // קבלת רשימה עם pagination וחיפוש
    static getByQuery = async (
        query: Partial<Template>,
        step: number,
        limit?: number
    ): Promise<TemplateDocument[]> => {
        return TemplateModel.find(
            query,
            {},
            limit ? { limit, skip: limit * step } : {}
        )
            .lean()  // מחזיר plain object במקום Mongoose Document
            .exec();
    };

    // ספירת מסמכים
    static getCount = async (query: Partial<Template>): Promise<number> => {
        return TemplateModel.countDocuments(query).lean().exec();
    };

    // קבלה לפי ID
    static getById = async (templateId: string): Promise<TemplateDocument> => {
        return TemplateModel.findById(templateId)
            .orFail(new DocumentNotFoundError(templateId))  // זורק שגיאה אם לא נמצא
            .lean()
            .exec();
    };

    // יצירה
    static createOne = async (template: Template): Promise<TemplateDocument> => {
        return TemplateModel.create(template);
    };

    // עדכון
    static updateOne = async (
        templateId: string,
        update: Partial<Template>
    ): Promise<TemplateDocument> => {
        return TemplateModel.findByIdAndUpdate(templateId, update, { new: true })
            .orFail(new DocumentNotFoundError(templateId))
            .lean()
            .exec();
    };

    // מחיקה
    static deleteOne = async (templateId: string): Promise<TemplateDocument> => {
        return TemplateModel.findByIdAndDelete(templateId)
            .orFail(new DocumentNotFoundError(templateId))
            .lean()
            .exec();
    };
}
```

**הסברים:**
- `.lean()` - מחזיר plain JavaScript object במקום Mongoose Document (ביצועים טובים יותר)
- `.orFail()` - זורק שגיאה אוטומטית אם לא נמצא מסמך
- `{ new: true }` - מחזיר את המסמך אחרי העדכון ולא לפני

---

#### `controller.ts` - HTTP Request Handlers

שכבת ה-Controller. מקבלת requests ומחזירה responses.

```typescript
import { Response } from 'express';
import { TypedRequest } from '../../utils/zod';
import { TemplateManager } from './manager';
import {
    createOneRequestSchema,
    deleteOneRequestSchema,
    getByIdRequestSchema,
    getByQueryRequestSchema,
    getCountRequestSchema,
    updateOneRequestSchema,
} from './validations';

export class TemplateController {
    static getByQuery = async (
        req: TypedRequest<typeof getByQueryRequestSchema>,
        res: Response
    ) => {
        const { step, limit, ...query } = req.query;
        res.json(await TemplateManager.getByQuery(query, step, limit));
    };

    static getCount = async (
        req: TypedRequest<typeof getCountRequestSchema>,
        res: Response
    ) => {
        res.json(await TemplateManager.getCount(req.query));
    };

    static getById = async (
        req: TypedRequest<typeof getByIdRequestSchema>,
        res: Response
    ) => {
        res.json(await TemplateManager.getById(req.params.id));
    };

    static createOne = async (
        req: TypedRequest<typeof createOneRequestSchema>,
        res: Response
    ) => {
        res.json(await TemplateManager.createOne(req.body));
    };

    static updateOne = async (
        req: TypedRequest<typeof updateOneRequestSchema>,
        res: Response
    ) => {
        res.json(await TemplateManager.updateOne(req.params.id, req.body));
    };

    static deleteOne = async (
        req: TypedRequest<typeof deleteOneRequestSchema>,
        res: Response
    ) => {
        res.json(await TemplateManager.deleteOne(req.params.id));
    };
}
```

**הסבר `TypedRequest`:**
- מקבל את ה-validation schema כ-generic
- מספק typing מלא ל-`req.body`, `req.params`, `req.query`
- IDE יודע בדיוק אילו שדות קיימים

---

#### `router.ts` - Feature Routes

הגדרת ה-Routes של ה-Feature.

```typescript
import { Router } from 'express';
import { validateRequest, wrapController } from '../../utils/express/wrappers';
import { TemplateController } from './controller';
import {
    createOneRequestSchema,
    deleteOneRequestSchema,
    getByIdRequestSchema,
    getByQueryRequestSchema,
    getCountRequestSchema,
    updateOneRequestSchema,
} from './validations';

export const templateRouter = Router();

templateRouter.get(
    '/',
    validateRequest(getByQueryRequestSchema),     // 1. וולידציה
    wrapController(TemplateController.getByQuery) // 2. Controller
);

templateRouter.get(
    '/count',
    validateRequest(getCountRequestSchema),
    wrapController(TemplateController.getCount)
);

templateRouter.get(
    '/:id',
    validateRequest(getByIdRequestSchema),
    wrapController(TemplateController.getById)
);

templateRouter.post(
    '/',
    validateRequest(createOneRequestSchema),
    wrapController(TemplateController.createOne)
);

templateRouter.put(
    '/:id',
    validateRequest(updateOneRequestSchema),
    wrapController(TemplateController.updateOne)
);

templateRouter.delete(
    '/:id',
    validateRequest(deleteOneRequestSchema),
    wrapController(TemplateController.deleteOne)
);
```

**הסבר:**
- `validateRequest` - מריץ וולידציה לפי ה-schema
- `wrapController` - עוטף את ה-controller ב-try/catch ומעביר שגיאות ל-error middleware

---

### תיקיית Utils

#### `utils/errors.ts` - Custom Error Classes

הגדרת שגיאות מותאמות אישית.

```typescript
// שגיאה בסיסית של השירות
export class ServiceError extends Error {
    constructor(
        public code: number,  // HTTP status code
        message: string,
    ) {
        super(message);
    }
}

// שגיאה כשמסמך לא נמצא
export class DocumentNotFoundError extends ServiceError {
    constructor(id: string) {
        super(404, `No feature found with id ${id}`);
    }
}
```

**הוספת שגיאות נוספות:**
```typescript
export class UnauthorizedError extends ServiceError {
    constructor(message = 'Unauthorized') {
        super(401, message);
    }
}

export class ForbiddenError extends ServiceError {
    constructor(message = 'Forbidden') {
        super(403, message);
    }
}

export class ValidationError extends ServiceError {
    constructor(message: string) {
        super(400, message);
    }
}

export class ConflictError extends ServiceError {
    constructor(message: string) {
        super(409, message);
    }
}
```

---

#### `utils/zod.ts` - Zod Utilities & Types

כלי עזר ל-Zod וטיפוסים ל-Express.

```typescript
import { Request } from 'express';
import { z, ZodObject, ZodRawShape } from 'zod';

// וולידציה של MongoDB ObjectId
export const zodMongoObjectId = z.string().regex(/^[0-9a-fA-F]{24}$/, {
    message: 'Invalid ObjectId',
});

// טיפוס בסיסי לסכמת Request
export type ReqSchema = ZodObject<{
    body: ZodObject<ZodRawShape>;
    query: ZodObject<ZodRawShape>;
    params: ZodObject<ZodRawShape>;
}>;

// חילוץ הטיפוס מהסכמה
export type SchemaOutput<T extends ReqSchema> = z.infer<T>;

// Helper להמרת unknown ל-object
type AsObj<T> = T extends object ? T : Record<string, never>;

// Request עם typing מלא לפי הסכמה
export type TypedRequest<T extends ReqSchema> = Request<
    AsObj<SchemaOutput<T>['params']>,
    unknown,
    AsObj<SchemaOutput<T>['body']>,
    AsObj<SchemaOutput<T>['query']>
>;
```

---

#### `utils/express/wrappers.ts` - Controller & Validation Wrappers

פונקציות עטיפה ל-controller ו-validation.

```typescript
import { NextFunction, Request, Response } from 'express';
import { ReqSchema, TypedRequest, SchemaOutput } from '../zod';

// עטיפה ל-middleware עם try/catch
const wrapMiddleware = (func: (req: Request, res?: Response) => Promise<void>) => {
    return async (req: Request, res: Response, next: NextFunction) => {
        try {
            await func(req, res);
            next();
        } catch (error) {
            next(error);
        }
    };
};

// עטיפה ל-controller עם try/catch
export const wrapController = <T extends ReqSchema>(
    fn: (req: TypedRequest<T>, res: Response, next?: NextFunction) => Promise<void>
) => {
    return async (req: Request, res: Response, next: NextFunction) => {
        try {
            await fn(req as unknown as TypedRequest<T>, res, next);
        } catch (error) {
            next(error);  // מעביר שגיאות ל-error middleware
        }
    };
};

// Middleware לוולידציה
export const validateRequest = <T extends ReqSchema>(schema: T) => {
    return wrapMiddleware(async (req: Request) => {
        // מריץ את הוולידציה על body, query, params
        const parsed: SchemaOutput<T> = await schema.parseAsync({
            body: (req.body ?? {}) as Record<string, unknown>,
            query: (req.query ?? {}) as Record<string, unknown>,
            params: (req.params ?? {}) as Record<string, unknown>,
        });

        // מחליף את הערכים המקוריים בערכים המפורסרים
        const r = req as unknown as TypedRequest<T>;
        r.body = parsed.body as typeof r.body;
        r.params = parsed.params as typeof r.params;

        try {
            r.query = parsed.query as typeof r.query;
        } catch {
            // Fallback אם req.query הוא readonly
            Object.defineProperty(r, 'query', {
                get: () => parsed.query,
                enumerable: true,
                configurable: true,
            });
        }
    });
};
```

---

#### `utils/express/error.ts` - Error Middleware

Middleware מרכזי לטיפול בשגיאות.

```typescript
import { NextFunction, Request, Response } from 'express';
import { ZodError } from 'zod';
import { fromZodError } from 'zod-validation-error';
import { ServiceError } from '../errors';

export const errorMiddleware = (
    error: Error,
    _req: Request,
    res: Response,
    next: NextFunction
) => {
    // שגיאת וולידציה מ-Zod
    if (error instanceof ZodError) {
        res.status(400).send({
            type: error.name,
            message: fromZodError(error).message,  // הודעה קריאה
        });
    }
    // שגיאה מותאמת של השירות
    else if (error instanceof ServiceError) {
        res.status(error.code).send({
            type: error.name,
            message: error.message,
        });
    }
    // שגיאה לא צפויה
    else {
        res.status(500).send({
            type: error.name,
            message: error.message,
        });
    }

    next();
};
```

**פורמט התגובה:**
```json
{
    "type": "ZodError",
    "message": "Validation error: email must be a valid email"
}
```

---

#### `utils/logger/index.ts` - Winston Logger

הגדרת ה-Logger.

```typescript
import * as winston from 'winston';

export const logger = winston.createLogger({
    transports: [new winston.transports.Console()],
    format: winston.format.combine(
        winston.format.colorize(),                    // צבעים בקונסול
        winston.format.timestamp(),                   // חותמת זמן
        winston.format.printf(({ level, message, timestamp }) =>
            `${String(timestamp)} [${String(level)}] ${String(message)}`
        ),
    ),
});
```

**שימוש:**
```typescript
import { logger } from './utils/logger';

logger.info('Server started');
logger.warn('Something might be wrong');
logger.error('An error occurred');
```

---

#### `utils/logger/middleware.ts` - Express Logger Middleware

Middleware ללוגים של requests.

```typescript
import * as expressWinston from 'express-winston';
import { logger } from './index';

export const loggerMiddleware = expressWinston.logger({
    transports: [logger],
    expressFormat: true,    // פורמט Express סטנדרטי
    statusLevels: true,     // לוגים לפי status code
});
```

---

## איך ליצור Feature חדש

### שלב 1: יצירת תיקיית ה-Feature

```
src/express/user/
├── interface.ts
├── model.ts
├── validations.ts
├── manager.ts
├── controller.ts
└── router.ts
```

### שלב 2: הגדרת ה-Interface

```typescript
// src/express/user/interface.ts
export interface User {
    firstName: string;
    lastName: string;
    email: string;
}

export interface UserDocument extends User {
    _id: string;
}
```

### שלב 3: יצירת ה-Model

```typescript
// src/express/user/model.ts
import mongoose from 'mongoose';
import { config } from '../../config';
import { UserDocument } from './interface';

const UserSchema = new mongoose.Schema<UserDocument>(
    {
        firstName: { type: String, required: true },
        lastName: { type: String, required: true },
        email: { type: String, required: true },
    },
    { versionKey: false },
);

export const UserModel = mongoose.model<UserDocument>('users', UserSchema);
```

### שלב 4: יצירת ה-Validations

```typescript
// src/express/user/validations.ts
import { z } from 'zod';
import { zodMongoObjectId } from '../../utils/zod';

const requiredFields = z.object({
    firstName: z.string().min(1),
    lastName: z.string().min(1),
    email: z.email(),
}).required();

export const createUserSchema = z.object({
    body: requiredFields,
    params: z.object({}),
    query: z.object({}),
});

export const getUserByIdSchema = z.object({
    body: z.object({}),
    params: z.object({ id: zodMongoObjectId }),
    query: z.object({}),
});

// ... שאר הסכמות
```

### שלב 5: יצירת ה-Manager

```typescript
// src/express/user/manager.ts
import { DocumentNotFoundError } from '../../utils/errors';
import { User, UserDocument } from './interface';
import { UserModel } from './model';

export class UserManager {
    static getById = async (userId: string): Promise<UserDocument> => {
        return UserModel.findById(userId)
            .orFail(new DocumentNotFoundError(userId))
            .lean()
            .exec();
    };

    static createOne = async (user: User): Promise<UserDocument> => {
        return UserModel.create(user);
    };

    // ... שאר הפונקציות
}
```

### שלב 6: יצירת ה-Controller

```typescript
// src/express/user/controller.ts
import { Response } from 'express';
import { TypedRequest } from '../../utils/zod';
import { UserManager } from './manager';
import { createUserSchema, getUserByIdSchema } from './validations';

export class UserController {
    static getById = async (
        req: TypedRequest<typeof getUserByIdSchema>,
        res: Response
    ) => {
        res.json(await UserManager.getById(req.params.id));
    };

    static createOne = async (
        req: TypedRequest<typeof createUserSchema>,
        res: Response
    ) => {
        res.json(await UserManager.createOne(req.body));
    };
}
```

### שלב 7: יצירת ה-Router

```typescript
// src/express/user/router.ts
import { Router } from 'express';
import { validateRequest, wrapController } from '../../utils/express/wrappers';
import { UserController } from './controller';
import { createUserSchema, getUserByIdSchema } from './validations';

export const userRouter = Router();

userRouter.get('/:id', validateRequest(getUserByIdSchema), wrapController(UserController.getById));
userRouter.post('/', validateRequest(createUserSchema), wrapController(UserController.createOne));
```

### שלב 8: חיבור ל-Router הראשי

```typescript
// src/express/router.ts
import { Router } from 'express';
import { templateRouter } from './template/router';
import { userRouter } from './user/router';  // הוספה

export const appRouter = Router();

appRouter.use('/api/template', templateRouter);
appRouter.use('/api/user', userRouter);  // הוספה

// ...
```

---

## Environment Variables

### `.env.development`

```env
PORT=8000
MONGO_URI=mongodb://localhost:27017/template-dev
TEMPLATE_SERVICE=templates
CORS_ORIGIN=http://localhost:3000
```

### `.env.production`

```env
PORT=8000
MONGO_URI=mongodb://mongo:27017/template-prod
TEMPLATE_SERVICE=templates
CORS_ORIGIN=https://your-domain.com
```

### משתנים נדרשים

| משתנה | תיאור | דוגמה |
|-------|-------|-------|
| `PORT` | פורט השרת | `8000` |
| `MONGO_URI` | Connection string ל-MongoDB | `mongodb://localhost:27017/mydb` |
| `TEMPLATE_SERVICE` | שם ה-collection | `templates` |
| `CORS_ORIGIN` | Origin מורשה ל-CORS | `http://localhost:3000` |

---

## סיכום

הטמפלייט הזה מספק בסיס איתן לפיתוח REST API עם:

1. **Type Safety מלא** - TypeScript strict mode + Zod validation
2. **ארכיטקטורה נקייה** - הפרדת שכבות ברורה
3. **טיפול בשגיאות** - מרוכז ועקבי
4. **לוגים** - מובנים עם Winston
5. **אבטחה** - Helmet + CORS
6. **פיתוח נוח** - Hot reload + ESLint + Prettier

כדי להשתמש בטמפלייט:
1. שכפל את הפרויקט
2. שנה את שם ה-Feature מ-`template` לשם המתאים
3. עדכן את ה-interface, model, validations לפי הצרכים
4. הוסף features נוספים לפי אותו מבנה
