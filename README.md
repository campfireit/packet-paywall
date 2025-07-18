
# Packet Paywall

A comprehensive captive portal management system designed for Managed Service Providers (MSPs) with multi-tenant architecture, payment integration, and Unifi controller support.

![Packet Paywall](https://img.shields.io/badge/Next.js-13+-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5+-blue?style=flat-square&logo=typescript)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-blue?style=flat-square&logo=postgresql)
![Prisma](https://img.shields.io/badge/Prisma-5+-2D3748?style=flat-square&logo=prisma)

## 🚀 Features

- **Multi-Tenant Architecture**: Complete MSP support with isolated client environments
- **Payment Integration**: Stripe integration for subscription and one-time payments
- **Unifi Controller Integration**: Seamless integration with Ubiquiti Unifi networks
- **Package Management**: Flexible internet package configuration and management
- **User Authentication**: Secure authentication with NextAuth.js
- **Transaction Management**: Complete payment and transaction tracking
- **Profile Management**: User profile and preference management
- **Responsive Design**: Modern, mobile-first UI with Tailwind CSS
- **Real-time Updates**: Live status updates and notifications

## 🛠️ Tech Stack

- **Frontend**: Next.js 13+ with App Router, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes, Prisma ORM
- **Database**: PostgreSQL
- **Authentication**: NextAuth.js
- **Payments**: Stripe
- **Network Integration**: Unifi Controller API
- **Deployment**: Vercel (recommended) or Docker

## 📋 Prerequisites

- Node.js 18+ 
- PostgreSQL 15+
- Stripe Account (for payments)
- Unifi Controller (for network management)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/campfireit/packet-paywall.git
cd packet-paywall
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Environment Setup

Copy the environment template and configure your variables:

```bash
cp .env.example .env.local
```

### 4. Database Setup

```bash
# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma migrate dev

# Seed the database (optional)
npx prisma db seed
```

### 5. Run Development Server

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to see the application.

## 🔧 Environment Variables

Create a `.env.local` file with the following variables:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/packet_paywall"

# NextAuth
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key"

# Stripe
STRIPE_PUBLISHABLE_KEY="pk_test_..."
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Unifi Controller
UNIFI_CONTROLLER_URL="https://your-controller.local:8443"
UNIFI_USERNAME="admin"
UNIFI_PASSWORD="password"

# Email (Optional)
EMAIL_SERVER_HOST="smtp.gmail.com"
EMAIL_SERVER_PORT=587
EMAIL_SERVER_USER="your-email@gmail.com"
EMAIL_SERVER_PASSWORD="your-app-password"
EMAIL_FROM="noreply@yourcompany.com"
```

## 🏗️ Project Structure

```
packet-paywall/
├── src/
│   ├── app/                 # Next.js App Router pages
│   ├── components/          # Reusable React components
│   ├── lib/                 # Utility functions and configurations
│   ├── types/               # TypeScript type definitions
│   └── styles/              # Global styles
├── prisma/
│   ├── schema.prisma        # Database schema
│   └── migrations/          # Database migrations
├── public/                  # Static assets
├── .github/
│   └── workflows/           # CI/CD workflows
└── docs/                    # Documentation
```

## 🚀 Deployment

### Vercel (Recommended)

1. **Connect to Vercel**:
   ```bash
   npm i -g vercel
   vercel login
   vercel
   ```

2. **Configure Environment Variables** in Vercel dashboard

3. **Set up Database**: Use Vercel Postgres or external PostgreSQL

4. **Deploy**:
   ```bash
   vercel --prod
   ```

### Docker Deployment

1. **Build Docker Image**:
   ```bash
   docker build -t packet-paywall .
   ```

2. **Run Container**:
   ```bash
   docker run -p 3000:3000 --env-file .env packet-paywall
   ```

### Manual Server Deployment

1. **Build Application**:
   ```bash
   npm run build
   ```

2. **Start Production Server**:
   ```bash
   npm start
   ```

## 🧪 Testing

```bash
# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

## 📚 API Documentation

The application provides RESTful APIs for:

- **Authentication**: `/api/auth/*`
- **Users**: `/api/users/*`
- **Packages**: `/api/packages/*`
- **Payments**: `/api/payments/*`
- **Transactions**: `/api/transactions/*`
- **Unifi Integration**: `/api/unifi/*`

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

For support and questions:

- Create an [Issue](https://github.com/campfireit/packet-paywall/issues)
- Check the [Documentation](./docs/)
- Contact: support@yourcompany.com

## 🙏 Acknowledgments

- [Next.js](https://nextjs.org/) - The React framework
- [Prisma](https://prisma.io/) - Database toolkit
- [Stripe](https://stripe.com/) - Payment processing
- [Tailwind CSS](https://tailwindcss.com/) - CSS framework
- [Ubiquiti](https://ui.com/) - Network hardware
