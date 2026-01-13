import React from 'react';
import { motion } from 'framer-motion';
import { Calendar, User, Percent, ChevronRight } from 'lucide-react';
import { Badge } from "@/components/ui/badge";
import { format, parseISO } from 'date-fns';
import { ptBR } from 'date-fns/locale';

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

export default function FlashCard({ task, onClick, index }) {
  const getStatusLabel = (progresso) => {
    if (progresso === 0) return { label: 'Não Iniciada', color: 'bg-slate-100 text-slate-700' };
    if (progresso === 100) return { label: 'Concluída', color: 'bg-emerald-100 text-emerald-700' };
    return { label: 'Em Andamento', color: 'bg-amber-100 text-amber-700' };
  };

  const status = getStatusLabel(task.progresso);
  const categoryColor = categoryColors[task.categoria] || '#6366f1';

  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ delay: index * 0.02 }}
      whileHover={{ scale: 1.02, y: -4 }}
      onClick={onClick}
      className="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden cursor-pointer hover:shadow-lg transition-all group"
    >
      {/* Color Bar */}
      <div 
        className="h-1.5"
        style={{ backgroundColor: categoryColor }}
      />
      
      <div className="p-5">
        {/* Header */}
        <div className="flex items-start justify-between gap-3 mb-3">
          <h3 className="font-semibold text-slate-800 text-sm leading-tight group-hover:text-indigo-700 transition-colors line-clamp-2">
            {task.tarefa}
          </h3>
          <ChevronRight className="h-4 w-4 text-slate-300 group-hover:text-indigo-500 transition-colors flex-shrink-0" />
        </div>

        {/* Category */}
        <p 
          className="text-xs font-medium mb-4 truncate"
          style={{ color: categoryColor }}
        >
          {task.categoria}
        </p>

        {/* Progress Bar */}
        <div className="mb-4">
          <div className="flex justify-between items-center mb-1.5">
            <span className="text-xs text-slate-500">Progresso</span>
            <span className="text-xs font-semibold text-slate-700">{task.progresso}%</span>
          </div>
          <div className="h-2 bg-slate-100 rounded-full overflow-hidden">
            <motion.div
              initial={{ width: 0 }}
              animate={{ width: `${task.progresso}%` }}
              transition={{ duration: 0.5, delay: index * 0.02 }}
              className="h-full rounded-full"
              style={{ backgroundColor: categoryColor }}
            />
          </div>
        </div>

        {/* Footer */}
        <div className="flex items-center justify-between pt-3 border-t border-slate-50">
          <div className="flex items-center gap-1.5 text-xs text-slate-500">
            <User className="h-3.5 w-3.5" />
            <span className="truncate max-w-[80px]">{task.responsavel}</span>
          </div>
          <Badge className={`${status.color} text-[10px] px-2 py-0.5`}>
            {status.label}
          </Badge>
        </div>

        {/* Dates */}
        <div className="flex items-center gap-1.5 mt-2 text-[10px] text-slate-400">
          <Calendar className="h-3 w-3" />
          {format(parseISO(task.inicio), 'dd/MM', { locale: ptBR })} - {format(parseISO(task.termino), 'dd/MM', { locale: ptBR })}
        </div>
      </div>
    </motion.div>
  );
}