# Comandos de Instalación - Dependencias de Testing

## 📦 Instalación Individual con Explicaciones

### 1. Jest - Framework de Testing Principal
**¿Qué hace?** Jest es el framework de testing más popular para JavaScript/TypeScript. Proporciona todo lo necesario para escribir y ejecutar pruebas: assertions, mocks, cobertura de código, etc.
```bash
npm install --save-dev jest@^30.2.0
```

### 2. TS-Jest - Transformador de TypeScript para Jest
**¿Qué hace?** Permite que Jest entienda y ejecute archivos TypeScript (.ts) directamente sin necesidad de compilarlos previamente a JavaScript.
```bash
npm install --save-dev ts-jest@^29.4.6
```

### 3. @types/jest - Definiciones de Tipos para Jest
**¿Qué hace?** Proporciona autocompletado y verificación de tipos en el IDE cuando escribes tests con Jest en TypeScript. Incluye los tipos para `describe`, `it`, `expect`, etc.
```bash
npm install --save-dev @types/jest@^30.0.0
```

### 4. Supertest - Testing de APIs HTTP
**¿Qué hace?** Librería especializada para hacer pruebas de endpoints HTTP. Permite simular peticiones (GET, POST, PUT, DELETE) a tu API sin necesidad de levantar un servidor real.
```bash
npm install --save-dev supertest@^7.1.4
```

### 5. @types/supertest - Definiciones de Tipos para Supertest
**¿Qué hace?** Proporciona tipado TypeScript para Supertest, permitiendo autocompletado y verificación de tipos al escribir tests de integración de APIs.
```bash
npm install --save-dev @types/supertest@^6.0.3
```

### 6. jest-mock-extended - Mocks Avanzados con TypeScript
**¿Qué hace?** Facilita la creación de mocks type-safe en TypeScript. Permite crear objetos simulados de interfaces y clases con verificación de tipos completa.
```bash
npm install --save-dev jest-mock-extended@^4.0.0
```

---

## 🚀 Instalación Rápida (Todas a la vez)
```bash
npm install --save-dev jest@^30.2.0 ts-jest@^29.4.6 @types/jest@^30.0.0 supertest@^7.1.4 @types/supertest@^6.0.3 jest-mock-extended@^4.0.0
```

---

## 📋 Resumen de Propósitos

| Paquete | Propósito Principal |
|---------|-------------------|
| `jest` | Framework base para ejecutar tests |
| `ts-jest` | Soporte TypeScript en Jest |
| `@types/jest` | Tipado TypeScript para Jest |
| `supertest` | Testing de endpoints HTTP/REST |
| `@types/supertest` | Tipado TypeScript para Supertest |
| `jest-mock-extended` | Mocks avanzados con tipos |

---

## ✅ Después de Instalar

1. Verificar instalación:
```bash
npm list --depth=0 | Select-String "jest|supertest"
```

2. Ejecutar tests:
```bash
npm test
```

3. Ver cobertura:
```bash
npm test -- --coverage
```
