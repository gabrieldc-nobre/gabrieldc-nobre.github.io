import React from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Badge } from "@/components/ui/badge";
import { motion } from 'framer-motion';
import { Calendar, User, Percent } from 'lucide-react';
import { format, parseISO } from 'date-fns';
import { ptBR } from 'date-fns/locale';

export default function TaskListModal({ open, onClose, title, tasks, onTaskClick }) {
  const getStatusColor = (progresso) => {
    if (progresso === 0) return 'bg-slate-100 text-slate-700';
    if (progresso === 100) return 'bg-emerald-100 text-emerald-700';
    return 'bg-amber-100 text-amber-700';
  };

  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-[700px] p-0 overflow-hidden">
        <DialogHeader className="px-6 pt-6 pb-4 bg-gradient-to-r from-indigo-50 to-white border-b border-slate-100">
          <DialogTitle className="text-xl font-semibold text-slate-800">
            {title}
            <span className="ml-2 text-sm font-normal text-slate-500">
              ({tasks.length} {tasks.length === 1 ? 'tarefa' : 'tarefas'})
            </span>
          </DialogTitle>
        </DialogHeader>
        
        <div className="p-4 max-h-[60vh] overflow-y-auto space-y-2">
          {tasks.map((task, idx) => (
            <motion.div
              key={task.id}
              initial={{ opacity: 0, y: 10 }}
              animate={{ opacity: 1, y: 0 }}
              transition={{ delay: idx * 0.03 }}
              onClick={() => onTaskClick(task)}
              className="p-4 rounded-xl border border-slate-100 hover:border-indigo-200 hover:bg-indigo-50/30 transition-all cursor-pointer group"
            >
              <div className="flex items-start justify-between gap-4">
                <div className="flex-1 min-w-0">
                  <h4 className="font-medium text-slate-800 group-hover:text-indigo-700 transition-colors truncate">
                    {task.tarefa}
                  </h4>
                  <p className="text-sm text-slate-500 mt-1 truncate">
                    {task.categoria}
                  </p>
                </div>
                <Badge className={getStatusColor(task.progresso)}>
                  {task.progresso}%
                </Badge>
              </div>
              
              <div className="flex items-center gap-4 mt-3 text-xs text-slate-500">
                <div className="flex items-center gap-1">
                  <User className="h-3.5 w-3.5" />
                  {task.responsavel}
                </div>
                <div className="flex items-center gap-1">
                  <Calendar className="h-3.5 w-3.5" />
                  {format(parseISO(task.inicio), 'dd/MM/yyyy', { locale: ptBR })} - {format(parseISO(task.termino), 'dd/MM/yyyy', { locale: ptBR })}
                </div>
              </div>
            </motion.div>
          ))}
          
          {tasks.length === 0 && (
            <div className="text-center py-12 text-slate-500">
              Nenhuma tarefa encontrada
            </div>
          )}
        </div>
      </DialogContent>
    </Dialog>
  );
}