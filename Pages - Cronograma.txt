import React, { useState, useEffect } from 'react';
import { base44 } from '@/api/base44Client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { addMonths, subMonths, parseISO } from 'date-fns';
import { Settings, FolderKanban, Users, FileSpreadsheet } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { toast } from "sonner";

import GanttChart from '../components/gantt/GanttChart';
import TaskModal from '../components/gantt/TaskModal';
import EditableTitle from '../components/common/EditableTitle';
import LogoUploader from '../components/common/LogoUploader';
import ManageCategoriesModal from '../components/settings/ManageCategoriesModal';
import ImportExportModal from '../components/import-export/ImportExportModal';

export default function Cronograma() {
  const queryClient = useQueryClient();
  const [currentDate, setCurrentDate] = useState(new Date(2026, 0, 1));
  const [selectedTask, setSelectedTask] = useState(null);
  const [showTaskModal, setShowTaskModal] = useState(false);
  const [showCategoriesModal, setShowCategoriesModal] = useState(false);
  const [showResponsiblesModal, setShowResponsiblesModal] = useState(false);
  const [showImportExportModal, setShowImportExportModal] = useState(false);

  // Queries
  const { data: tasks = [] } = useQuery({
    queryKey: ['tasks'],
    queryFn: () => base44.entities.Task.list()
  });

  const { data: categories = [] } = useQuery({
    queryKey: ['categories'],
    queryFn: () => base44.entities.Category.list()
  });

  const { data: responsibles = [] } = useQuery({
    queryKey: ['responsibles'],
    queryFn: () => base44.entities.Responsible.list()
  });

  const { data: settings } = useQuery({
    queryKey: ['settings'],
    queryFn: async () => {
      const list = await base44.entities.AppSettings.list();
      return list[0] || null;
    }
  });

  // Mutations
  const createTask = useMutation({
    mutationFn: (data) => base44.entities.Task.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries(['tasks']);
      toast.success('Tarefa criada com sucesso!');
    }
  });

  const updateTask = useMutation({
    mutationFn: ({ id, data }) => base44.entities.Task.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries(['tasks']);
      toast.success('Tarefa atualizada!');
    }
  });

  const deleteTask = useMutation({
    mutationFn: (id) => base44.entities.Task.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries(['tasks']);
      toast.success('Tarefa excluída!');
    }
  });

  const createCategory = useMutation({
    mutationFn: (nome) => base44.entities.Category.create({ nome }),
    onSuccess: () => {
      queryClient.invalidateQueries(['categories']);
      toast.success('Categoria adicionada!');
    }
  });

  const updateCategory = useMutation({
    mutationFn: async ({ id, nome, oldName }) => {
      await base44.entities.Category.update(id, { nome });
      // Update all tasks with this category
      const tasksToUpdate = tasks.filter(t => t.categoria === oldName);
      for (const task of tasksToUpdate) {
        await base44.entities.Task.update(task.id, { categoria: nome });
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['categories']);
      queryClient.invalidateQueries(['tasks']);
      toast.success('Categoria atualizada!');
    }
  });

  const deleteCategory = useMutation({
    mutationFn: (id) => base44.entities.Category.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries(['categories']);
      toast.success('Categoria removida!');
    }
  });

  const createResponsible = useMutation({
    mutationFn: (nome) => base44.entities.Responsible.create({ nome }),
    onSuccess: () => {
      queryClient.invalidateQueries(['responsibles']);
      toast.success('Responsável adicionado!');
    }
  });

  const updateResponsible = useMutation({
    mutationFn: async ({ id, nome, oldName }) => {
      await base44.entities.Responsible.update(id, { nome });
      // Update all tasks with this responsible
      const tasksToUpdate = tasks.filter(t => t.responsavel === oldName);
      for (const task of tasksToUpdate) {
        await base44.entities.Task.update(task.id, { responsavel: nome });
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['responsibles']);
      queryClient.invalidateQueries(['tasks']);
      toast.success('Responsável atualizado!');
    }
  });

  const deleteResponsible = useMutation({
    mutationFn: (id) => base44.entities.Responsible.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries(['responsibles']);
      toast.success('Responsável removido!');
    }
  });

  const updateSettings = useMutation({
    mutationFn: async (data) => {
      if (settings?.id) {
        return base44.entities.AppSettings.update(settings.id, data);
      } else {
        return base44.entities.AppSettings.create(data);
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['settings']);
    }
  });

  // Handlers
  const handleTaskClick = (task) => {
    setSelectedTask(task);
    setShowTaskModal(true);
  };

  const handleAddTask = () => {
    setSelectedTask(null);
    setShowTaskModal(true);
  };

  const handleSaveTask = (data, taskId) => {
    if (taskId) {
      updateTask.mutate({ id: taskId, data });
    } else {
      createTask.mutate(data);
    }
    setShowTaskModal(false);
    setSelectedTask(null);
  };

  const handleDeleteTask = (taskId) => {
    deleteTask.mutate(taskId);
    setShowTaskModal(false);
    setSelectedTask(null);
  };

  const handleUpdateCategoryName = (id, newName) => {
    const cat = categories.find(c => c.id === id);
    if (cat) {
      updateCategory.mutate({ id, nome: newName, oldName: cat.nome });
    }
  };

  const handleUpdateResponsibleName = (id, newName) => {
    const resp = responsibles.find(r => r.id === id);
    if (resp) {
      updateResponsible.mutate({ id, nome: newName, oldName: resp.nome });
    }
  };

  // Import/Export handlers
  const handleImport = async (file) => {
    const { file_url } = await base44.integrations.Core.UploadFile({ file });
    const result = await base44.integrations.Core.ExtractDataFromUploadedFile({
      file_url,
      json_schema: {
        type: "array",
        items: {
          type: "object",
          properties: {
            tarefa: { type: "string" },
            categoria: { type: "string" },
            detalhamento: { type: "string" },
            responsavel: { type: "string" },
            observacoes: { type: "string" },
            progresso: { type: "number" },
            inicio: { type: "string" },
            termino: { type: "string" }
          }
        }
      }
    });

    if (result.status === 'success' && result.output) {
      for (const task of result.output) {
        await base44.entities.Task.create({
          ...task,
          progresso: typeof task.progresso === 'string' 
            ? parseInt(task.progresso.replace('%', '')) 
            : (task.progresso || 0)
        });
      }
      queryClient.invalidateQueries(['tasks']);
    }
  };

  const handleReplace = async (file) => {
    // Delete all existing tasks
    for (const task of tasks) {
      await base44.entities.Task.delete(task.id);
    }
    await handleImport(file);
  };

  const handleExport = async () => {
    const csvContent = [
      ['TAREFA', 'CATEGORIA', 'DETALHAMENTO', 'RESPONSÁVEL', 'OBSERVAÇÕES', 'PROGRESSO', 'INÍCIO', 'TÉRMINO'].join('\t'),
      ...tasks.map(t => [
        t.tarefa,
        t.categoria,
        t.detalhamento || '',
        t.responsavel,
        t.observacoes || '',
        `${t.progresso}%`,
        t.inicio,
        t.termino
      ].join('\t'))
    ].join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'tarefas_projeto.csv';
    link.click();
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 via-white to-indigo-50/30 p-4 md:p-8">
      <div className="max-w-[1600px] mx-auto">
        {/* Header */}
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
          <div className="flex items-center gap-4">
            <LogoUploader 
              logoUrl={settings?.logo_url} 
              onLogoChange={(url) => updateSettings.mutate({ logo_url: url })}
            />
            <EditableTitle 
              value={settings?.titulo_cronograma || 'Cronograma do Projeto'} 
              onChange={(value) => updateSettings.mutate({ titulo_cronograma: value })}
            />
          </div>
          
          <div className="flex items-center gap-2 flex-wrap">
            <Button
              variant="outline"
              onClick={() => setShowCategoriesModal(true)}
              className="rounded-xl gap-2 border-slate-200 hover:bg-slate-50"
            >
              <FolderKanban className="h-4 w-4" />
              Categorias
            </Button>
            <Button
              variant="outline"
              onClick={() => setShowResponsiblesModal(true)}
              className="rounded-xl gap-2 border-slate-200 hover:bg-slate-50"
            >
              <Users className="h-4 w-4" />
              Responsáveis
            </Button>
            <Button
              variant="outline"
              onClick={() => setShowImportExportModal(true)}
              className="rounded-xl gap-2 border-slate-200 hover:bg-slate-50"
            >
              <FileSpreadsheet className="h-4 w-4" />
              Excel
            </Button>
          </div>
        </div>

        {/* Gantt Chart */}
        <GanttChart
          tasks={tasks}
          categories={categories}
          onTaskClick={handleTaskClick}
          onAddTask={handleAddTask}
          currentDate={currentDate}
          onPrevMonth={() => setCurrentDate(prev => subMonths(prev, 1))}
          onNextMonth={() => setCurrentDate(prev => addMonths(prev, 1))}
        />

        {/* Modals */}
        <TaskModal
          open={showTaskModal}
          onClose={() => setShowTaskModal(false)}
          task={selectedTask}
          categories={categories}
          responsibles={responsibles}
          onSave={handleSaveTask}
          onDelete={handleDeleteTask}
        />

        <ManageCategoriesModal
          open={showCategoriesModal}
          onClose={() => setShowCategoriesModal(false)}
          items={categories}
          title="Gerenciar Categorias"
          onAdd={(nome) => createCategory.mutate(nome)}
          onUpdate={handleUpdateCategoryName}
          onDelete={(id) => deleteCategory.mutate(id)}
        />

        <ManageCategoriesModal
          open={showResponsiblesModal}
          onClose={() => setShowResponsiblesModal(false)}
          items={responsibles}
          title="Gerenciar Responsáveis"
          onAdd={(nome) => createResponsible.mutate(nome)}
          onUpdate={handleUpdateResponsibleName}
          onDelete={(id) => deleteResponsible.mutate(id)}
        />

        <ImportExportModal
          open={showImportExportModal}
          onClose={() => setShowImportExportModal(false)}
          onImport={handleImport}
          onReplace={handleReplace}
          onExport={handleExport}
        />
      </div>
    </div>
  );
}