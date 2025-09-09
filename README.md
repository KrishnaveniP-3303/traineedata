using Microsoft.AspNetCore.Mvc;
using TraineeData.Models;
using TraineeData.Services;

namespace TraineeData.Controllers
{
    public class TraineesController : Controller
    {
        private readonly TraineeService _service;

        public TraineesController()
        {
            _service = new TraineeService();
        }

        public IActionResult Index()
        {
            var trainees = _service.GetAll();
            return View(trainees);
        }

        public IActionResult Create()
        {
            return View();
        }

        [HttpPost]
        public IActionResult Create(Trainee trainee)
        {
            if (ModelState.IsValid)
            {
                _service.Add(trainee);
                return RedirectToAction("Index");
            }
            return View(trainee);
        }

        public IActionResult Edit(int id)
        {
            var trainee = _service.GetById(id);
            if (trainee == null)
                return NotFound();

            return View(trainee);
        }

        [HttpPost]
        public IActionResult Edit(Trainee trainee)
        {
            if (ModelState.IsValid)
            {
                _service.Update(trainee);
                return RedirectToAction("Index");
            }
            return View(trainee);
        }

        public IActionResult Delete(int id)
        {
            var trainee = _service.GetById(id);
            if (trainee == null)
                return NotFound();

            return View(trainee);
        }

        [HttpPost, ActionName("Delete")]
        public IActionResult DeleteConfirmed(int id)
        {
            _service.Delete(id);
            return RedirectToAction("Index");
        }
    }
}
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text.Json;
using TraineeData.Models;

namespace TraineeData.Services
{
    public class TraineeService
    {
        private readonly string _filePath = Path.Combine(Directory.GetCurrentDirectory(), "App_Data", "trainees.json");

        public List<Trainee> GetAll()
        {
            if (!File.Exists(_filePath))
                return new List<Trainee>();

            var json = File.ReadAllText(_filePath);
            return JsonSerializer.Deserialize<List<Trainee>>(json) ?? new List<Trainee>();
        }

        public void SaveAll(List<Trainee> trainees)
        {
            var json = JsonSerializer.Serialize(trainees, new JsonSerializerOptions { WriteIndented = true });
            File.WriteAllText(_filePath, json);
        }

        public void Add(Trainee trainee)
        {
            var trainees = GetAll();
            trainee.Id = trainees.Any() ? trainees.Max(t => t.Id) + 1 : 1;
            trainees.Add(trainee);
            SaveAll(trainees);
        }

        public Trainee GetById(int id)
        {
            return GetAll().FirstOrDefault(t => t.Id == id);
        }

        public void Update(Trainee updatedTrainee)
        {
            var trainees = GetAll();
            var index = trainees.FindIndex(t => t.Id == updatedTrainee.Id);
            if (index != -1)
            {
                trainees[index] = updatedTrainee;
                SaveAll(trainees);
            }
        }

        public void Delete(int id)
        {
            var trainees = GetAll();
            var traineeToRemove = trainees.FirstOrDefault(t => t.Id == id);
            if (traineeToRemove != null)
            {
                trainees.Remove(traineeToRemove);
                SaveAll(trainees);
            }
        }
    }
}
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - TraineeData</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/TraineeData.styles.css" asp-append-version="true" />

    <style>
        body {
            background-color: #f1f7ff;
        }

        .sidebar {
            min-height: 100vh;
          background-color: rgb(135, 206, 235);
            box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
        }

        .sidebar .nav-link.active {
            font-weight: bold;
            color:;
        }
    </style>
</head>

<body>
   
    <nav class="navbar navbar-light bg-light d-md-none">
        <div class="container-fluid">
            <button class="btn btn-outline-primary d-flex align-items-center" type="button"
    data-bs-toggle="collapse" data-bs-target="#sidebarMenu"
    aria-controls="sidebarMenu" aria-expanded="false" aria-label="Toggle navigation">
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor"
        class="bi bi-list me-2" viewBox="0 0 16 16">
        <path fill-rule="evenodd"
            d="M2.5 12.5a.5.5 0 0 1 0-1h11a.5.5 0 0 1 0 1h-11zm0-4a.5.5 0 0 1 0-1h11a.5.5 0 0 1 0 1h-11zm0-4a.5.5 0 0 1 0-1h11a.5.5 0 0 1 0 1h-11z" />
    </svg>
    Menu
</button>
   <span class="navbar-brand mb-0 h1">TraineeData</span>
        </div>
    </nav>

    <div class="d-flex">
       
        <nav class="col-md-2 sidebar collapse d-md-block p-3" id="sidebarMenu">
            <div class="text-center mb-4">
                <img src="/images/download.jpeg" class="rounded-circle w-50" alt="Profile">
                <h5 class="mt-2">Peter</h5>
            </div>
            <ul class="nav flex-column">
                <li class="nav-item mb-2">
                    <a class="nav-link active" asp-controller="Home" asp-action="Index">Dashboard</a>
                </li>
                <li class="nav-item mb-2">
                    <a class="nav-link" asp-controller="Trainees" asp-action="Index">
                        Trainees
                    </a>
                </li>
                <li class="nav-item mb-2">
                    <a class="nav-link" asp-controller="Libraries" asp-action="Index">Libraries</a>
                </li>
                <li class="nav-item mb-2">
                    <a class="nav-link" asp-controller="Saved" asp-action="Index">Saved</a>
                </li>
            </ul>
        </nav>

        <!-- Main Content -->
        <div class="container-fluid p-4" style="background-color: rgba(203, 234, 246, 1);">
            <main role="main" class="pb-3">
                @RenderBody()
            </main>
        </div>
    </div>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>

</html>
@model List<TraineeData.Models.Trainee>

@{
    ViewData["Title"] = "Trainees List";
}

<div class="container my-5">

    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1 class="h3 text-primary">Trainees List</h1>
        <a asp-controller="Trainees" asp-action="Create" class="btn btn-primary">
            ➕ Add New Trainee
        </a>
    </div>

    <div class="card shadow-sm">
        <div class="card-body">
            <table class="table table-hover table-striped align-middle">
                <thead class="table-primary">
                    <tr>
                        <th>Name</th>
                        <th>DOB</th>
                        <th>Gender</th>
                        <th>Degree</th>
                        <th class="text-center">Actions</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach (var trainee in Model)
                    {
                        <tr>
                            <td>@trainee.Name</td>
                            <td>@trainee.Dob.ToShortDateString()</td>
                            <td>@trainee.Gender</td>
                            <td>@trainee.Degree</td>
                            <td class="text-center">
                                <a asp-action="Edit" asp-route-id="@trainee.Id" class="btn btn-sm btn-primary me-2">
                                    ✏️ Edit
                                </a>
                                <a asp-action="Delete" asp-route-id="@trainee.Id" class="btn btn-sm btn-danger">
                                    ️ Delete
                                </a>
                            </td>
                        </tr>
                    }
                </tbody>
            </table>

            @if (!Model.Any())
            {
                <div class="alert alert-info text-center mt-4">
                    No trainees available. Click "Add New Trainee" to get started.
                </div>
            }
        </div>
    </div>

</div>
@model TraineeData.Models.Trainee

<h2 class="mb-4 text-center text-primary">Create New Trainee</h2>

<div class="row justify-content-center">
    <div class="col-lg-8">
        <div class="card shadow-sm border-0">
            <div class="card-body">
                <form asp-action="Create" method="post" class="row g-3">
                    <div class="col-md-6">
                        <label asp-for="BioId" class="form-label fw-bold">Bio ID</label>
                        <input asp-for="BioId" class="form-control" placeholder="Enter Bio ID" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Name" class="form-label fw-bold">Name</label>
                        <input asp-for="Name" class="form-control" placeholder="Enter Name" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Dob" class="form-label fw-bold">Date of Birth</label>
                        <input asp-for="Dob" type="date" class="form-control" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Gender" class="form-label fw-bold">Gender</label>
                        <select asp-for="Gender" class="form-select">
                            <option value="">Select Gender</option>
                            <option value="Male">Male</option>
                            <option value="Female">Female</option>
                        </select>
                    </div>

                    <div class="col-md-6">
                        <label asp-for="DoJ" class="form-label fw-bold">Date of Joining</label>
                        <input asp-for="DoJ" type="date" class="form-control" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="CurrentDate" class="form-label fw-bold">Current Date</label>
                        <input asp-for="CurrentDate" type="date" class="form-control" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Degree" class="form-label fw-bold">Degree</label>
                        <input asp-for="Degree" class="form-control" placeholder="Enter Degree" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Reference" class="form-label fw-bold">Reference</label>
                        <input asp-for="Reference" class="form-control" placeholder="Enter Reference" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Software" class="form-label fw-bold">Software</label>
                        <input asp-for="Software" class="form-control" placeholder="Enter Software" />
                    </div>

                    <div class="col-md-6">
                        <label asp-for="Months" class="form-label fw-bold">Months</label>
                        <input asp-for="Months" type="number" class="form-control" placeholder="Enter Months" />
                    </div>

                    <div class="col-12">
                        <label asp-for="Learning" class="form-label fw-bold">Learning</label>
                        <input asp-for="Learning" class="form-control" placeholder="What is the trainee learning?" />
                    </div>

                    <div class="col-12 text-center mt-4">
                        <button type="submit" class="btn btn-primary me-2 px-4">Save</button>
                        <a asp-action="Index" class="btn btn-outline-secondary px-4">Cancel</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
