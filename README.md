# 🧪 API Automation Test with Postman & Newman

## 📘 Overview
This project API testing using **Postman** and **Newman**.  
It includes automated execution, JSON Schema validation, and HTML reporting.

---

## ⚙️ Prerequisites

- Node.js ≥ v14  
- npm  
- Postman  
- Newman

Install globally:
```bash
npm install -g newman newman-reporter-htmlextra
```

Or install locally (recommended):
```bash
npm install newman newman-reporter-htmlextra
```

---

## 📁 Project Structure
```
📦 API automation project folder/
├── API_Example_Collection.json
│   └── Validate an object/
│       └── TC001_Create an object success
│       └── TC002_Get an object by Id
│       └── TC003_Get an invalid object by Id
├── API_Example_Env/
│   └── api_env.json
├── reports/
│   └── API_Test_Report.html
├── README.md
```

---

## ▶️ Run Tests

### Run and generate HTML report
```bash
npx newman run API_Example_Collection.postman_collection.json \
  -e API_Example_Env.postman_environment.json \
  -r htmlextra \
  --reporter-htmlextra-export ./reports/API_Test_Report.html \
  --reporter-htmlextra-title "API Test Report"
```

---

## 🧾 Example Test in Postman
```javascript
const schema = {
  type: "object",
  required: ["id", "name", "data"],
  properties: {
    id: { type: "string" },
    name: { type: "string" },
    data: {
      type: "object",
      properties: {
        year: { type: "number" },
        price: { type: "number" },
        "CPU model": { type: "string" },
        "Hard disk size": { type: "string" }
      },
      required: ["year", "price", "CPU model", "Hard disk size"]
    }
  }
};

pm.test("Response matches schema", () => {
  pm.response.to.have.jsonSchema(schema);
});
```

## 📊 Output Example
After successful execution, the report will be available at:
```
./reports/API_Test_Report.html
```

Open it in your browser to view:
- ✅ Test pass/fail summary  
- ⏱ Response times  
- 📦 Detailed request/response logs  

---

## ✅ Summary

| Task | Command |
|------|----------|
| Run Tests | `npx newman run ...` |
| Run with Environment | `-e environments/api_env.json` |
| Generate HTML Report | `-r htmlextra --reporter-htmlextra-export ./reports/report.html` |
