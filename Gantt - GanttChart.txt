import React, { useState, useMemo } from 'react';
import { ChevronLeft, ChevronRight, Plus, Edit2, Check, X } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { motion } from 'framer-motion';
import { format, startOfMonth, endOfMonth, eachDayOfInterval, isSameMonth, isWithinInterval, parseISO, differenceInDays } from 'date-fns';
import { ptBR } from 'date-fns/locale';
import { cn } from "@/lib/utils";

const categoryColors = {
  'Definição dos Objetivos Gerenciais': '#6366f1',
  'Estruturação dos Centros de Custos': '#8b5cf6',
  'Definição de Papéis e Responsabilidades': '#ec4899',
  'Estruturação da Área Comercial': '#f59e0b',
  'Estruturação da Área de Marketing': '#10b981',
  'Estruturação Administrativa e Apoio': '#06b6d4',
  'Definição de Indicadores de Desempenho': '#3b82f6',
  'Definição dos Critérios de Rateio': '#ef4444',
  'Diagnóstico Inicial': '#84cc16',
  'Organização e Parametrização': '#f97316',
  'Teste Piloto do Modelo': '#14b8a6',
  'Implantação Definitiva': '#a855f7',
  'Acompanhamento e Melhoria Contínua': '#0ea5e9',
  'Avaliação dos Resultados': '#22c55e',
};

export default function GanttChart({ 
  tasks, 
  categories, 
  onTaskClick, 
  onAddTask,
  currentDate,
  onPrevMonth,
  onNextMonth 
}) {
  const monthStart = startOfMonth(currentDate);
  const monthEnd = endOfMonth(currentDate);
  const daysInMonth = eachDayOfInterval({ start: monthStart, end: monthEnd });

  const tasksByCategory = useMemo(() => {
    const grouped = {};
    categories.forEach(cat => {
      grouped[cat.nome] = tasks.filter(t => t.categoria === cat.nome);
    });
    return grouped;
  }, [tasks, categories]);

  const getTaskPosition = (task) => {
    const taskStart = parseISO(task.inicio);
    const taskEnd = parseISO(task.termino);
    
    const visibleStart = taskStart < monthStart ? monthStart : taskStart;
    const visibleEnd = taskEnd > monthEnd ? monthEnd : taskEnd;
    
    if (visibleStart > monthEnd || visibleEnd < monthStart) {
      return null;
    }

    const startDay = differenceInDays(visibleStart, monthStart);
    const duration = differenceInDays(visibleEnd, visibleStart) + 1;
    const totalDays = daysInMonth.length;

    return {
      left: `${(startDay / totalDays) * 100}%`,
      width: `${(duration / totalDays) * 100}%`,
      startsBeforeMonth: taskStart < monthStart,
      endsAfterMonth: taskEnd > monthEnd
    };
  };

  const getProgressColor = (progress) => {
    if (progress === 0) return 'bg-slate-200';
    if (progress < 50) return 'bg-amber-400';
    if (progress < 100) return 'bg-blue-500';
    return 'bg-emerald-500';
  };

  return (
    <div className="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
      {/* Header com navegação */}
      <div className="flex items-center justify-between px-6 py-4 border-b border-slate-100 bg-gradient-to-r from-slate-50 to-white">
        <div className="flex items-center gap-3">
          <Button 
            variant="ghost" 
            size="icon" 
            onClick={onPrevMonth}
            className="h-9 w-9 rounded-xl hover:bg-slate-100"
          >
            <ChevronLeft className="h-5 w-5 text-slate-600" />
          </Button>
          <h3 className="text-lg font-semibold text-slate-800 min-w-[180px] text-center capitalize">
            {format(currentDate, 'MMMM yyyy', { locale: ptBR })}
          </h3>
          <Button 
            variant="ghost" 
            size="icon" 
            onClick={onNextMonth}
            className="h-9 w-9 rounded-xl hover:bg-slate-100"
          >
            <ChevronRight className="h-5 w-5 text-slate-600" />
          </Button>
        </div>
        <Button 
          onClick={onAddTask}
          className="bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl gap-2"
        >
          <Plus className="h-4 w-4" />
          Nova Tarefa
        </Button>
      </div>

      {/* Grid de dias */}
      <div className="flex border-b border-slate-100">
        <div className="w-72 min-w-72 bg-slate-50/50 p-3 text-xs font-medium text-slate-500 uppercase tracking-wider border-r border-slate-100 sticky left-0 z-10">
          Categoria / Tarefa
        </div>
        <div className="flex-1 flex">
          {daysInMonth.map((day, idx) => (
            <div 
              key={idx} 
              className={cn(
                "flex-1 min-w-8 p-2 text-center text-xs border-r border-slate-50 last:border-r-0",
                day.getDay() === 0 || day.getDay() === 6 ? 'bg-slate-50/50 text-slate-400' : 'text-slate-500'
              )}
            >
              <div className="font-medium">{format(day, 'd')}</div>
              <div className="text-[10px] uppercase">{format(day, 'EEE', { locale: ptBR })}</div>
            </div>
          ))}
        </div>
      </div>

      {/* Conteúdo */}
      <div className="max-h-[600px] overflow-auto">
        {categories.map((category, catIdx) => (
          <div key={category.id || catIdx}>
            {/* Categoria Header - Sticky */}
            <div className="flex sticky top-0 z-20 bg-white border-b border-slate-100">
              <div 
                className="w-72 min-w-72 p-3 font-semibold text-sm sticky left-0 bg-white border-r border-slate-100 flex items-center gap-2"
                style={{ color: categoryColors[category.nome] || '#6366f1' }}
              >
                <div 
                  className="w-2.5 h-2.5 rounded-full"
                  style={{ backgroundColor: categoryColors[category.nome] || '#6366f1' }}
                />
                {category.nome}
                <span className="text-slate-400 font-normal text-xs ml-auto">
                  ({tasksByCategory[category.nome]?.length || 0})
                </span>
              </div>
              <div className="flex-1 bg-slate-50/30" />
            </div>

            {/* Tarefas da categoria */}
            {tasksByCategory[category.nome]?.map((task, taskIdx) => {
              const position = getTaskPosition(task);
              
              return (
                <motion.div 
                  key={task.id}
                  initial={{ opacity: 0, y: 10 }}
                  animate={{ opacity: 1, y: 0 }}
                  transition={{ delay: taskIdx * 0.02 }}
                  className="flex border-b border-slate-50 hover:bg-slate-50/50 transition-colors group"
                >
                  <div 
                    className="w-72 min-w-72 p-3 text-sm text-slate-700 border-r border-slate-100 sticky left-0 bg-white group-hover:bg-slate-50/50 transition-colors cursor-pointer flex items-center gap-2"
                    onClick={() => onTaskClick(task)}
                  >
                    <div 
                      className="w-1.5 h-1.5 rounded-full flex-shrink-0"
                      style={{ backgroundColor: categoryColors[category.nome] || '#6366f1' }}
                    />
                    <span className="truncate">{task.tarefa}</span>
                    <Edit2 className="h-3.5 w-3.5 text-slate-400 opacity-0 group-hover:opacity-100 transition-opacity ml-auto flex-shrink-0" />
                  </div>
                  <div className="flex-1 relative h-12 flex items-center">
                    {position && (
                      <div
                        className={cn(
                          "absolute h-7 rounded-lg overflow-hidden cursor-pointer transition-all hover:h-8 hover:shadow-md",
                          position.startsBeforeMonth && 'rounded-l-none',
                          position.endsAfterMonth && 'rounded-r-none'
                        )}
                        style={{ 
                          left: position.left, 
                          width: position.width,
                          backgroundColor: `${categoryColors[category.nome] || '#6366f1'}20`
                        }}
                        onClick={() => onTaskClick(task)}
                      >
                        <div 
                          className={cn("h-full transition-all", getProgressColor(task.progresso))}
                          style={{ 
                            width: `${task.progresso}%`,
                            backgroundColor: task.progresso > 0 ? categoryColors[category.nome] || '#6366f1' : undefined
                          }}
                        />
                        <div className="absolute inset-0 flex items-center justify-center">
                          <span className="text-[10px] font-medium text-slate-700 px-2 truncate">
                            {task.progresso}%
                          </span>
                        </div>
                      </div>
                    )}
                  </div>
                </motion.div>
              );
            })}
          </div>
        ))}
      </div>
    </div>
  );
}