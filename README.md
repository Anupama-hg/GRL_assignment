# GRL_assignment

```
using Microsoft.EntityFrameworkCore;
using TaskHub.Data;
using TaskHub.Enums;
using TaskHub.Models;
using TaskHub.Repositories.Interfaces;

namespace TaskHub.Repositories.Implementations
{
    public class TaskRepository : ITaskRepository
    {
        private readonly AppDbContext _context;

        public TaskRepository(AppDbContext context)
        {
            _context = context;
        }

        public async Task<IEnumerable<TaskItem>> GetAllTasksAsync()
        {
            return await _context.tasks.Include(t => t.user).ToListAsync();
        }

        public async Task<TaskItem?> GetTaskByIdAsync(int id)
        {
            return await _context.tasks.Include(t => t.user).FirstOrDefaultAsync(t => t.id == id);
        }

        public async Task<IEnumerable<TaskItem>> GetTasksByUserIdAsync(int userId)
        {
            return await _context.tasks
                .Where(t => t.userid == userId)
                .Include(t => t.user)
                .ToListAsync();
        }

        public async Task<TaskItem> AddTaskAsync(TaskItem task)
        {
            _context.tasks.Add(task);
            await _context.SaveChangesAsync();
            return task;
        }

        public async Task<TaskItem?> UpdateTaskAsync(TaskItem task)
        {
            var existingTask = await _context.tasks.FindAsync(task.id);
            if (existingTask == null) return null;

            existingTask.title = task.title;
            existingTask.description = task.description;
            existingTask.priority = task.priority;
            existingTask.duedate = task.duedate;
            existingTask.userid = task.userid;

            await _context.SaveChangesAsync();
            return existingTask;
        }

        public async Task<bool> DeleteTaskAsync(int id)
        {
            var task = await _context.tasks.FindAsync(id);
            if (task == null) return false;

            _context.tasks.Remove(task);
            await _context.SaveChangesAsync();
            return true;
        }

        public async Task<IEnumerable<TaskItem>> FilterTasksAsync(TaskStatus? status, TaskPriority? priority, DateTime? startDueDate, DateTime? endDueDate)
        {
            var query = _context.tasks.AsQueryable();

            if (status.HasValue)
                query = query.Where(t => t.status == status.Value);

            if (priority.HasValue)
                query = query.Where(t => t.priority == priority.Value);

            if (startDueDate.HasValue)
                query = query.Where(t => t.duedate >= startDueDate.Value);

            if (endDueDate.HasValue)
                query = query.Where(t => t.duedate <= endDueDate.Value);

            return await query.Include(t => t.user).ToListAsync();
        }

        public async Task<IEnumerable<TaskItem>> SortTasksAsync(string sortBy)
        {
            IQueryable<TaskItem> query = _context.tasks.Include(t => t.user);

            return sortBy.ToLower() switch
            {
                "duedate" => await query.OrderBy(t => t.duedate).ToListAsync(),
                "priority" => await query.OrderBy(t => t.priority).ToListAsync(),
                _ => await query.ToListAsync(),
            };
        }
    }
}

```


```

usb_pd_parser.py

Parses the Table of Contents (ToC) from a USB PD specification PDF file
and outputs a structured JSONL file (usb_pd_spec.jsonl).

Usage:
    python usb_pd_parser.py input.pdf
"""

import sys
import re
import json
import fitz  # PyMuPDF

DOC_TITLE = "USB Power Delivery Specification Rev X"

def extract_toc_lines(pdf_path, max_pages=10):
    """Extract candidate ToC lines from the first few pages."""
    toc_lines = []
    with fitz.open(pdf_path) as doc:
        for page in doc[:max_pages]:
            text = page.get_text()
            for line in text.split('\n'):
                if re.match(r'^\d+(\.\d+)*\s+.+\s+\d+$', line.strip()):
                    toc_lines.append(line.strip())
    return toc_lines

def parse_toc_line(line):
    """
    Parse ToC line into structured fields.
    Example line: "2.1.2 Power Delivery Contract Negotiation ........ 53"
    """
    match = re.match(r'^(\d+(\.\d+)*)(\s+)(.+?)\s+\.{2,}\s+(\d+)$', line)
    if not match:
        return None
    section_id = match.group(1)
    title = match.group(4).strip()
    page = int(match.group(5))
    level = section_id.count('.') + 1
    parent_id = '.'.join(section_id.split('.')[:-1]) if '.' in section_id else None
    full_path = f"{section_id} {title}"
    return {
        "doc_title": DOC_TITLE,
        "section_id": section_id,
        "title": title,
        "full_path": full_path,
        "page": page,
        "level": level,
        "parent_id": parent_id,
        "tags": []
    }

def parse_pdf_toc(pdf_path, output_path='usb_pd_spec.jsonl'):
    """Main parser function to extract and write JSONL."""
    toc_lines = extract_toc_lines(pdf_path)
    with open(output_path, 'w', encoding='utf-8') as fout:
        for line in toc_lines:
            parsed = parse_toc_line(line)
            if parsed:
                fout.write(json.dumps(parsed) + '\n')
    print(f"✅ Parsed {len(toc_lines)} lines and saved to {output_path}")

if _name_ == "_main_":
    if len(sys.argv) != 2:
        print("Usage: python usb_pd_parser.py input.pdf")
    else:
        parse_pdf_toc(sys.argv[1])

```
