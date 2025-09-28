# Project 1: Personal Portfolio Website
*3 weeks | Frontend fundamentals & basic backend*

## Project Overview
Build a personal portfolio website using React, TypeScript, and Node.js. This project focuses on frontend fundamentals, basic backend API development, and creating a complete full-stack application that showcases your skills and projects.

## Learning Objectives
- Master React and TypeScript fundamentals
- Learn Node.js and Express for backend development
- Implement basic CRUD operations
- Understand full-stack application architecture
- Practice responsive web design
- Learn about API integration

## Technical Requirements

### **Frontend Technologies**
- **Framework**: React 18+
- **Language**: TypeScript
- **Styling**: CSS Modules / Styled Components
- **State Management**: React Context / Redux Toolkit
- **Routing**: React Router
- **HTTP Client**: Axios

### **Backend Technologies**
- **Runtime**: Node.js 18+
- **Framework**: Express
- **Database**: MongoDB
- **ORM**: Mongoose
- **Authentication**: JWT
- **Validation**: Joi / Yup

### **Project Structure**
```
portfolio-website/
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── context/
│   │   ├── services/
│   │   ├── types/
│   │   └── utils/
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── middleware/
│   │   ├── services/
│   │   └── utils/
│   ├── package.json
│   └── tsconfig.json
├── shared/
│   ├── types/
│   └── utils/
├── docker-compose.yml
├── README.md
└── .gitignore
```

## Week 1: Frontend Fundamentals & React Setup

### **Day 1-2: React & TypeScript Basics**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: React and TypeScript concepts
- **8:00 PM - 9:00 PM**: Set up React project
- **9:00 PM - 9:30 PM**: Review and practice

**Learning Content:**
- React components and JSX
- TypeScript basics and type safety
- Props and state management
- Event handling
- Component lifecycle

**Implementation:**
- Set up React with TypeScript
- Create basic components
- Implement state management
- Add routing

**Resources:**
- [React Documentation](https://react.dev/) - Official docs
- [TypeScript Handbook](https://www.typescriptlang.org/docs/) - TypeScript reference
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) - TypeScript + React

### **Day 3-4: Component Development**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Component design patterns
- **8:00 PM - 9:00 PM**: Build portfolio components
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Component composition
- Custom hooks
- Context API
- Styling with CSS Modules
- Responsive design

**Implementation:**
- Create header and navigation
- Build hero section
- Implement about section
- Add projects showcase

**Resources:**
- [React Hooks](https://react.dev/reference/react) - Hooks reference
- [CSS Modules](https://github.com/css-modules/css-modules) - CSS Modules
- [Responsive Design](https://web.dev/responsive-web-design-basics/) - Responsive design

### **Day 5: State Management & API Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: State management concepts
- **8:00 PM - 9:00 PM**: Implement API integration
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Context API for state management
- API integration with Axios
- Error handling
- Loading states
- Data fetching patterns

**Implementation:**
- Set up API service
- Implement data fetching
- Add error handling
- Create loading states

**Resources:**
- [Axios Documentation](https://axios-http.com/docs/intro) - HTTP client
- [React Context](https://react.dev/learn/passing-data-deeply-with-context) - Context API
- [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) - Error handling

## Week 2: Backend Development & API

### **Day 1-2: Node.js & Express Setup**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Node.js and Express concepts
- **8:00 PM - 9:00 PM**: Set up backend project
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Node.js runtime environment
- Express framework basics
- Middleware and routing
- Error handling
- Environment configuration

**Implementation:**
- Set up Express server
- Create basic routes
- Implement middleware
- Add error handling

**Resources:**
- [Node.js Documentation](https://nodejs.org/docs/) - Runtime reference
- [Express Documentation](https://expressjs.com/) - Framework reference
- [Express Middleware](https://expressjs.com/en/guide/using-middleware.html) - Middleware

### **Day 3-4: Database Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: MongoDB and Mongoose concepts
- **8:00 PM - 9:00 PM**: Implement database integration
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- MongoDB database design
- Mongoose ODM
- Schema design
- CRUD operations
- Data validation

**Implementation:**
- Set up MongoDB
- Create data models
- Implement CRUD operations
- Add data validation

**Resources:**
- [MongoDB Documentation](https://docs.mongodb.com/) - Database reference
- [Mongoose Documentation](https://mongoosejs.com/docs/) - ODM reference
- [MongoDB Schema Design](https://docs.mongodb.com/manual/core/data-modeling-introduction/) - Schema design

### **Day 5: Authentication & Security**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Authentication concepts
- **8:00 PM - 9:00 PM**: Implement authentication
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- JWT authentication
- Password hashing
- User registration and login
- Protected routes
- Security best practices

**Implementation:**
- Create user model
- Implement authentication
- Add protected routes
- Set up security middleware

**Resources:**
- [JWT Documentation](https://jwt.io/introduction/) - Token authentication
- [bcrypt](https://www.npmjs.com/package/bcrypt) - Password hashing
- [Express Security](https://expressjs.com/en/advanced/best-practice-security.html) - Security practices

## Week 3: Full-Stack Integration & Deployment

### **Day 1-2: API Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: API integration concepts
- **8:00 PM - 9:00 PM**: Connect frontend and backend
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- RESTful API design
- HTTP methods and status codes
- API error handling
- CORS configuration
- API documentation

**Implementation:**
- Connect frontend to backend
- Implement API calls
- Add error handling
- Configure CORS

**Resources:**
- [REST API Design](https://restfulapi.net/) - API design
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) - Status codes
- [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) - Cross-origin requests

### **Day 3-4: Testing & Quality Assurance**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Testing concepts
- **8:00 PM - 9:00 PM**: Implement comprehensive tests
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Unit testing with Jest
- Integration testing
- End-to-end testing
- Test coverage
- Mocking strategies

**Implementation:**
- Write unit tests
- Create integration tests
- Add end-to-end tests
- Set up test coverage

**Resources:**
- [Jest Documentation](https://jestjs.io/docs/getting-started) - Testing framework
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) - React testing
- [Supertest](https://www.npmjs.com/package/supertest) - API testing

### **Day 5: Deployment & Production**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Deployment concepts
- **8:00 PM - 9:00 PM**: Deploy to production
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Docker containerization
- Cloud deployment
- Environment configuration
- CI/CD pipeline
- Monitoring and logging

**Implementation:**
- Create Docker containers
- Deploy to cloud platform
- Set up CI/CD
- Configure monitoring

**Resources:**
- [Docker Documentation](https://docs.docker.com/) - Containerization
- [Vercel](https://vercel.com/) - Frontend deployment
- [Railway](https://railway.app/) - Backend deployment

## Frontend Components

### **Header Component**
- Navigation menu
- Logo and branding
- Mobile responsive menu
- User authentication status

### **Hero Section**
- Personal introduction
- Call-to-action buttons
- Background image/video
- Animated elements

### **About Section**
- Personal story
- Skills and expertise
- Professional experience
- Education background

### **Projects Showcase**
- Project cards
- Project details modal
- Filter and search
- Live demo links

### **Contact Section**
- Contact form
- Social media links
- Contact information
- Map integration

## Backend API Endpoints

### **Authentication**
```
POST   /api/auth/register    - User registration
POST   /api/auth/login       - User login
POST   /api/auth/logout      - User logout
GET    /api/auth/me          - Get current user
```

### **Portfolio Management**
```
GET    /api/portfolios       - Get all portfolios
POST   /api/portfolios       - Create portfolio
GET    /api/portfolios/:id   - Get portfolio by ID
PUT    /api/portfolios/:id   - Update portfolio
DELETE /api/portfolios/:id   - Delete portfolio
```

### **Projects Management**
```
GET    /api/projects         - Get all projects
POST   /api/projects         - Create project
GET    /api/projects/:id     - Get project by ID
PUT    /api/projects/:id     - Update project
DELETE /api/projects/:id     - Delete project
```

## Database Schema

### **Users Collection**
```javascript
{
  _id: ObjectId,
  email: String,
  passwordHash: String,
  firstName: String,
  lastName: String,
  bio: String,
  avatarUrl: String,
  createdAt: Date,
  updatedAt: Date
}
```

### **Portfolios Collection**
```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  title: String,
  description: String,
  websiteUrl: String,
  githubUrl: String,
  isPublic: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

### **Projects Collection**
```javascript
{
  _id: ObjectId,
  portfolioId: ObjectId,
  title: String,
  description: String,
  technologies: [String],
  githubUrl: String,
  demoUrl: String,
  imageUrl: String,
  createdAt: Date,
  updatedAt: Date
}
```

## Testing Strategy

### **Frontend Tests**
- Component unit tests
- Hook testing
- Integration tests
- User interaction tests

### **Backend Tests**
- API endpoint tests
- Database operation tests
- Authentication tests
- Error handling tests

### **End-to-End Tests**
- Complete user workflows
- Cross-browser testing
- Performance testing
- Accessibility testing

### **Test Coverage Goals**
- **Minimum**: 80% code coverage
- **Target**: 90% code coverage
- **Focus**: Critical business logic

## Deployment Checklist

### **Pre-deployment**
- [ ] All tests passing
- [ ] Code review completed
- [ ] Documentation updated
- [ ] Environment variables configured
- [ ] Database migrations ready

### **Production Setup**
- [ ] Docker containers built
- [ ] Database configured
- [ ] SSL certificates installed
- [ ] Monitoring configured
- [ ] Backup strategy implemented

### **Post-deployment**
- [ ] Health checks passing
- [ ] Performance monitoring active
- [ ] Error tracking configured
- [ ] Documentation accessible
- [ ] Team notified

## Success Criteria

### **Frontend Requirements**
- [ ] Responsive and accessible UI
- [ ] Clean, maintainable React code
- [ ] Comprehensive test coverage (90%+)
- [ ] Proper error handling
- [ ] Performance optimization

### **Backend Requirements**
- [ ] Clean, maintainable Node.js code
- [ ] Well-designed API endpoints
- [ ] Proper database schema design
- [ ] Security best practices
- [ ] Performance optimization

### **Full-Stack Integration**
- [ ] Seamless frontend-backend communication
- [ ] Proper error handling across layers
- [ ] Consistent user experience
- [ ] Scalable architecture
- [ ] Production-ready deployment

## Next Steps

1. **Complete Project 1** with all requirements
2. **Deploy to production** and test thoroughly
3. **Document lessons learned** and challenges
4. **Prepare for Project 2** (Blog Management Platform)
5. **Continue learning** full-stack development

Ready to build your personal portfolio website? Let's start coding! 🚀
