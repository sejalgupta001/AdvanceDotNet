# Implementing File Upload, Update, and Deletion 

- **Adding a File** (during meeting creation).
- **Updating a File** (replacing an existing file during meeting update).
- **Deleting a File** (removing the file when deleting a meeting or independently).

## Step 1: Set Up Project Dependencies and Configurations
#### In `Program.cs ` :
```csharp

// File service registration
builder.Services.AddScoped<IFileService, FileService>();

// Limit file upload size (e.g., 15MB) to prevent abuse
builder.Services.Configure<FormOptions>(options =>
{
    options.MultipartBodyLengthLimit = 15 * 1024 * 1024; // 15 MiB
});

var app = builder.Build();

// Enable static files for serving uploads
app.UseStaticFiles();

```

**Points to Note**:
- `MultipartBodyLengthLimit` prevents large file uploads; adjust as needed.

## Step 2: Define Models and DTOs
#### In `Meeting DTO` :
```csharp
namespace MinutesOfMeeting.Models
{
    // DTO 
  public class Meeting
{
    public int? MeetingID { get; set; }
    public DateTime MeetingDate { get; set; }
    public int MeetingVenueID { get; set; }
    public int MeetingTypeID { get; set; }
    public int DepartmentID { get; set; }
    public string? MeetingTitle { get; set; }
    public string? MeetingNumber { get; set; }
    public string? MeetingDescription { get; set; }
    public string? DocumentPath { get; set; }
    public bool? IsCancelled { get; set; }
    public IFormFile? DocumentFile { get; set; }
    public DateTime? CancellationDateTime { get; set; }
    public string? CancellationReason { get; set; }
}
}
```

## Step 3: Implement the File Service (IFileService)
**Explanation**: Handles upload, deletion, and validation. Upload generates unique names; delete cleans up disk space.

**Code Snippet** (In `Services/FileService.cs`):
```csharp
namespace MinutesOfMeeting.Services
{
    public interface IFileService
    {
        Task<string> UploadFileAsync(IFormFile file, string subFolder);
        void DeleteFile(string? relativePath);
    }

    public class FileService : IFileService
    {
        private readonly string _basePath;
        private readonly string _filesFolder = "Files";

        public FileService()
        {
            _basePath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", _filesFolder);
        }

        public async Task<string> UploadFileAsync(IFormFile file, string subFolder)
        {
            if (file == null || file.Length == 0)
                throw new ArgumentException("File is empty");

            // Validate file type (optional but recommended)
            var allowedExtensions = new[] { ".jpg", ".jpeg", ".png", ".pdf" };
            var extension = Path.GetExtension(file.FileName).ToLowerInvariant();
            if (!allowedExtensions.Contains(extension))
                throw new ArgumentException($"Invalid file type. Allowed: {string.Join(", ", allowedExtensions)}");

            // Validate size (redundant with FormOptions but good practice)
            if (file.Length > 15 * 1024 * 1024)
                throw new ArgumentException("File exceeds 15MB");

            var folderPath = Path.Combine(_basePath, subFolder);
            if (!Directory.Exists(folderPath))
                Directory.CreateDirectory(folderPath);

            var fileName = $"{Guid.NewGuid()}_{file.FileName}";
            var filePath = Path.Combine(folderPath, fileName);

            using (var stream = new FileStream(filePath, FileMode.Create))
            {
                await file.CopyToAsync(stream);
            }

            return Path.Combine(_filesFolder, subFolder, fileName); // Relative path for DB
        }

        public void DeleteFile(string? relativePath)
        {
            if (string.IsNullOrEmpty(relativePath)) return;

            var fullPath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", relativePath);
            if (File.Exists(fullPath))
            {
                try
                {
                    File.Delete(fullPath);
                }
                catch (Exception ex)
                {
                    // Log error (e.g., use ILogger); don't throw to avoid blocking operations
                    Console.WriteLine($"Failed to delete file: {fullPath}. Error: {ex.Message}");
                }
            }
        }
    }
}
```

## Step 4: Implement Controller Endpoints
**Explanation**: Update `MOM_MeetingController` for Create (add file), Update (replace file), and Delete (remove file).

**Code Snippet** (In `Controllers/MOM_MeetingController.cs`):
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using MinutesOfMeeting.Models;
using MinutesOfMeeting.Services;

namespace MinutesOfMeeting.Controllers
{
    [Route("api/[controller]/[action]")]
    [ApiController]
    public class MOM_MeetingController : BaseApiController
    {
        private readonly ApplicationDbContext _db;
        private readonly IFileService _fileService;

        public MOM_MeetingController(ApplicationDbContext db, IFileService fileService)
        {
            _db = db;
            _fileService = fileService;
        }

        // CREATE (Add File)
        [HttpPost]
        public async Task<IActionResult> Create([FromForm] MeetingCreateDto model)
        {
            if (!ModelState.IsValid)
            {
                var errors = ModelState.Values.SelectMany(v => v.Errors).Select(e => e.ErrorMessage).ToList();
                return BadRequest(ValidationErrorResponse(errors));
            }

            string? filePath = null;
            if (model.DocumentFile != null)
            {
                filePath = await _fileService.UploadFileAsync(model.DocumentFile, "Meetings");
            }

            var meeting = new MOM_Meeting
            {
                MeetingDate = model.MeetingDate,
                MeetingVenueID = model.MeetingVenueID,
                MeetingTypeID = model.MeetingTypeID,
                DepartmentID = model.DepartmentID,
                MeetingTitle = model.MeetingTitle,
                MeetingNumber = model.MeetingNumber,
                MeetingDescription = model.MeetingDescription,
                DocumentPath = filePath, //Add path
                IsCancelled = model.IsCancelled,
                CancellationDateTime = model.CancellationDateTime,
                CancellationReason = model.CancellationReason,
                Created = DateTime.Now,
                Modified = DateTime.Now
            };

            await _db._Meetings.AddAsync(meeting);
            await _db.SaveChangesAsync();
            return Ok(SuccessResponse("Meeting created successfully", meeting));
        }

        // UPDATE (Update/Replace File)
        [HttpPut("{id:int}")]
        public async Task<IActionResult> Update(int id, [FromForm] MeetingUpdateDto model)
        {
            if (!ModelState.IsValid)
            {
                var errors = ModelState.Values.SelectMany(v => v.Errors).Select(e => e.ErrorMessage).ToList();
                return BadRequest(ValidationErrorResponse(errors));
            }

            if (id != model.MeetingID)
                return BadRequest(ErrorResponse("IDs do not match"));

            var existing = await _db._Meetings.FindAsync(id);
            if (existing == null)
                return NotFound(ErrorResponse("Meeting not found"));

            // File handling: Replace if new file provided
            if (model.DocumentFile != null && model.DocumentFile.Length > 0)
            {
                // Delete old file
                _fileService.DeleteFile(existing.DocumentPath);

                // Upload new
                existing.DocumentPath = await _fileService.UploadFileAsync(model.DocumentFile, "Meetings");
            }

            // Update other fields
            existing.MeetingDate = model.MeetingDate;
            existing.MeetingVenueID = model.MeetingVenueID;
            existing.MeetingTypeID = model.MeetingTypeID;
            existing.DepartmentID = model.DepartmentID;
            existing.MeetingTitle = model.MeetingTitle;
            existing.MeetingNumber = model.MeetingNumber;
            existing.MeetingDescription = model.MeetingDescription;
            existing.IsCancelled = model.IsCancelled;
            existing.CancellationDateTime = model.CancellationDateTime;
            existing.CancellationReason = model.CancellationReason;
            existing.Modified = DateTime.Now;

            _db._Meetings.Update(existing);
            await _db.SaveChangesAsync();
            return Ok(SuccessResponse("Meeting updated successfully", existing));
        }

        // DELETE (Delete File)
        [HttpDelete("{id:int}")]
        public async Task<IActionResult> Delete(int id)
        {
            var meeting = await _db._Meetings.FindAsync(id);
            if (meeting == null)
                return NotFound(ErrorResponse("Meeting not found"));

            // Delete associated file
            _fileService.DeleteFile(meeting.DocumentPath);

            _db._Meetings.Remove(meeting);
            await _db.SaveChangesAsync();
            return Ok(SuccessResponse("Meeting deleted successfully"));
        }

        // Other methods (GetAll, GetById, etc.) 
    }
}
```

**Points to Note**:
- Use `[FromForm]` for multipart requests in Create/Update.
- Delete only happens if path exists; no error if missing.
- Modification: Add a separate `/api/MOM_Meeting/DeleteFile/{id}` endpoint for file-only deletion without removing meeting.
