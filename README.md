const express = require("express");
const mongoose = require("mongoose");
const app = express();

app.use(express.json());

// ✅ MongoDB Connection
mongoose.connect("mongodb://127.0.0.1:27017/studentDB", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB Connected"))
.catch((err) => console.log(err));

// ✅ Model (Student)
const studentSchema = new mongoose.Schema({
  name: { type: String, required: true },
  age: { type: Number, required: true },
  course: { type: String, required: true },
});

const Student = mongoose.model("Student", studentSchema);

// ✅ Controller Functions
const studentController = {
  createStudent: async (req, res) => {
    try {
      const student = new Student(req.body);
      await student.save();
      res.status(201).json(student);
    } catch (err) {
      res.status(400).json({ message: err.message });
    }
  },

  getAllStudents: async (req, res) => {
    try {
      const students = await Student.find();
      res.json(students);
    } catch (err) {
      res.status(500).json({ message: err.message });
    }
  },

  getStudentById: async (req, res) => {
    try {
      const student = await Student.findById(req.params.id);
      if (!student) return res.status(404).json({ message: "Student not found" });
      res.json(student);
    } catch (err) {
      res.status(500).json({ message: err.message });
    }
  },

  updateStudent: async (req, res) => {
    try {
      const student = await Student.findByIdAndUpdate(req.params.id, req.body, { new: true });
      if (!student) return res.status(404).json({ message: "Student not found" });
      res.json(student);
    } catch (err) {
      res.status(400).json({ message: err.message });
    }
  },

  deleteStudent: async (req, res) => {
    try {
      const student = await Student.findByIdAndDelete(req.params.id);
      if (!student) return res.status(404).json({ message: "Student not found" });
      res.json({ message: "Student deleted successfully" });
    } catch (err) {
      res.status(500).json({ message: err.message });
    }
  },
};

// ✅ Routes
app.post("/api/students", studentController.createStudent);
app.get("/api/students", studentController.getAllStudents);
app.get("/api/students/:id", studentController.getStudentById);
app.put("/api/students/:id", studentController.updateStudent);
app.delete("/api/students/:id", studentController.deleteStudent);

// ✅ Server Start
const PORT = 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
